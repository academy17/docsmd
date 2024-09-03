---
description: Functions related to Account Management
---

# Account Facet

The `AccountFacetImpl.sol` library manages account-related operations within the SYMM contracts, focusing on deposits, withdrawals, and allocation of funds. It interacts with several storage structures to manage account balances, ensure compliance with cooldown periods, and handle liquidation statuses.

The `AccountFacet.sol` handles the emitting of events related to the implementation's functions.

### How allocation works in our system

After depositing collateral to the SYMM Diamond, the user cannot trade with it yet. It is required to allocate that money to a subaccount, then the user can start trading. Sub-accounts are isolated instances, as SYMMIO is 100% economically sound, all `PartyA` <> `PartyB` instances are isolated in subaccounts. Collateral deposited into a subaccount is in cross with all positions opened using that subaccount. To have an isolated position, a seperate subaccount should be created. Subaccounts also enable further customization down the road, where collateral could be allocated to a specific contract instance, with customized code written by `PartyA` or `PartyB`.

When a user allocates, they specify a fraction of, or the entire deposited amount to engage in trading. This specification is factored in when assessing the user's liquidity status, regardless of the total amount the user has deposited. When a user deallocates, they return a portion, or even the entirety, of their allocation back into their deposits. A withdrawal cooldown is applied to deallocated funds.

## **PartyA Function Overview**

### **deposit()**

**Description**: This function allows users to deposit tokens into the SYMM Diamond, to be allocated to sub-accounts for trading.&#x20;

The designated token to serve as collateral is predetermined following the deployment of the contract, in current SYMMIO version each contract has a unique collateral type, for multi collateral support multiple base contracts can be deployed. It is important to note that whatever design decision gets taken in later versions, it is critical to hard require both Parties to use the same collateral when entering into a trade, in this SYMMIO version that hard requirement is enforced by having a single collateral class per contract. (see the [`ControlFacet.setCollateral`](control-facet.md#setcollateral) function).

There's also the functionality for a user to deposit funds on behalf of another user utilizing the `depositFor` method.

```solidity
function deposit(uint256 amount) external whenNotAccountingPaused {
        AccountFacetImpl.deposit(msg.sender, amount);
        emit Deposit(msg.sender, msg.sender, amount);
    }
```

**Implementation:**

```solidity
    function deposit(address user, uint256 amount) internal {
        GlobalAppStorage.Layout storage appLayout = GlobalAppStorage.layout();
        IERC20(appLayout.collateral).safeTransferFrom(msg.sender, address(this), amount);
        uint256 amountWith18Decimals = (amount * 1e18) /
        (10 ** IERC20Metadata(appLayout.collateral).decimals());
        AccountStorage.layout().balances[user] += amountWith18Decimals;
    }
```

**Input Parameters**:&#x20;

* `user`: the address of the user.
* `amount`: the amount of tokens to deposit to the user.

**Storage Interactions**: Updates the user's balance in `AccountStorage`.

**Event Structure:**

```solidity
event Deposit(address sender, address user, uint256 amount);
```

### **withdraw()**

**Description**: Enables users to withdraw tokens from their account after a cooldown period. This collateral must first be de-allocated before it can be withdrawn.

Funds that have been deposited but not yet allocated can be returned to the user's wallet. Additionally, there's a required waiting period between the deallocation and withdrawal processes. The waiting period can and will be used for independent watch dogs and security researcher to detect potential malicious behaviour between PartyA and PartyBs and can therefore be used to suspend those malicious parties, suspended Parties are not able to withdraw their de-allocated funds after the withdrawal period. It's a typical Fraud Proof window also used in optimistic rollups, one could therefore describe SYMMIO as something vaguely similiar to an L3.

```solidity
function withdraw(uint256 amount) external whenNotAccountingPaused notSuspended(msg.sender) {
        AccountFacetImpl.withdraw(msg.sender, amount);
        emit Withdraw(msg.sender, msg.sender, amount);
    }
```

**Implementation:**

```solidity
    function withdraw(address user, uint256 amount) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        GlobalAppStorage.Layout storage appLayout = GlobalAppStorage.layout();
        require(
            block.timestamp >=
            accountLayout.withdrawCooldown[msg.sender] + MAStorage.layout().deallocateCooldown,
            "AccountFacet: Cooldown hasn't reached"
        );
        uint256 amountWith18Decimals = (amount * 1e18) /
        (10 ** IERC20Metadata(appLayout.collateral).decimals());
        accountLayout.balances[msg.sender] -= amountWith18Decimals;
        IERC20(appLayout.collateral).safeTransfer(user, amount);
    }
```

**Input Parameters**:&#x20;

* `user`: the address of the user.deposi
* `amount`: the amount of tokens to deposit to the user.

**Key Validations**: Ensures the cooldown period is respected using `MAStorage`.

**Storage Interactions**: Decreases the user's balance via `accountLayout.balances` in `AccountStorage`.

**Event Structure:**

```solidity
event Withdraw(address sender, address user, uint256 amount);
```

### **allocate()**

**Description**: Allocates a specified amount of funds to an account.

A user can specify a fraction of, or the entire deposited amount to engage in trading. This specification is factored in when assessing the user's liquidity status, regardless of the total amount the user has deposited.

```solidity
    function allocate(
        uint256 amount
    ) external whenNotAccountingPaused notLiquidatedPartyA(msg.sender) {
        AccountFacetImpl.allocate(amount);
        emit AllocatePartyA(msg.sender, amount);
    }
```

**Implementation:**

```solidity
   function allocate(uint256 amount) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        require(
            accountLayout.allocatedBalances[msg.sender] + amount <=
            GlobalAppStorage.layout().balanceLimitPerUser,
            "AccountFacet: Allocated balance limit reached"
        );
        require(accountLayout.balances[msg.sender] >= amount, "AccountFacet: Insufficient balance");
        accountLayout.balances[msg.sender] -= amount;
        accountLayout.allocatedBalances[msg.sender] += amount;
    }
```

**Input Parameters**:&#x20;

* `amount`: the amount of tokens to deposit to the user.

**Storage Interactions**: Modifies the `allocatedBalances` mapping in `AccountStorage`.

**Event Structure:**

```solidity
event AllocatePartyA(address user, uint256 amount);
```

### **depositAndAllocate()**

**Description:** Combines the `deposit()` and `allocate()` functions into one function for user convenience.

```solidity
    function depositAndAllocate(
        uint256 amount
    ) external whenNotAccountingPaused notLiquidatedPartyA(msg.sender) {
        AccountFacetImpl.deposit(msg.sender, amount);
        uint256 amountWith18Decimals = (amount * 1e18) /
            (10 ** IERC20Metadata(GlobalAppStorage.layout().collateral).decimals());
        AccountFacetImpl.allocate(amountWith18Decimals);
        emit Deposit(msg.sender, msg.sender, amount);
        emit AllocatePartyA(msg.sender, amountWith18Decimals);
    }
```

**Event Structure:**

```solidity
    event Deposit(address sender, address user, uint256 amount);
    event AllocatePartyA(address user, uint256 amount);
```

### **deallocate()**

**Description**: Deallocates funds previously allocated to a user account.

If not utilized elsewhere, users have the option to return a portion, or even the entirety, of their allocation back into their deposits.

```solidity
    function deallocate(
        uint256 amount,
        SingleUpnlSig memory upnlSig
    ) external whenNotAccountingPaused notLiquidatedPartyA(msg.sender) {
        AccountFacetImpl.deallocate(amount, upnlSig);
        emit DeallocatePartyA(msg.sender, amount);
    }
```

**Implementation:**

```solidity
    function deallocate(uint256 amount, SingleUpnlSig memory upnlSig) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        require(
            accountLayout.allocatedBalances[msg.sender] >= amount,
            "AccountFacet: Insufficient allocated Balance"
        );
        LibMuon.verifyPartyAUpnl(upnlSig, msg.sender);
        int256 availableBalance = LibAccount.partyAAvailableForQuote(upnlSig.upnl, msg.sender);
        require(availableBalance >= 0, "AccountFacet: Available balance is lower than zero");
        require(uint256(availableBalance) >= amount, "AccountFacet: partyA will be liquidatable");

        accountLayout.partyANonces[msg.sender] += 1;
        accountLayout.allocatedBalances[msg.sender] -= amount;
        accountLayout.balances[msg.sender] += amount;
        accountLayout.withdrawCooldown[msg.sender] = block.timestamp;
    }
```

**Input Parameters**:&#x20;

* `amount`: the amount of tokens to deposit to the user.
* `upnlSig`: a cryptographic signature obtained from Muon to verify user upnl. This is a factor in determining whether the user has enough capital to `deallocate()` without facing liquidation.

**Verifications**: Verifies the signature using `LibMuon.verifyPartyAUpnl`, then assesses if the user's `availableBalance` exceeds the amount they're requesting to deallocate.

**Storage Interactions**: Adjusts allocated and free balances, updates nonces and sets the cooldown start time in `AccountStorage`.

**Event Structure:**

```solidity
    event DeallocatePartyA(address user, uint256 amount);
```

## **PartyB Function Overview**

### **transferAllocation()**

**Description**: Transfers allocated funds from one `partyB` to another, ensuring both parties meet solvency requirements.

```solidity
    function transferAllocation(
        uint256 amount,
        address origin,
        address recipient,
        SingleUpnlSig memory upnlSig
    ) external whenNotPartyBActionsPaused {
        AccountFacetImpl.transferAllocation(amount, origin, recipient, upnlSig);
        emit TransferAllocation(amount, origin, recipient);
    }
```

**Implementation:**

```solidity
    function transferAllocation(
        uint256 amount,
        address origin,
        address recipient,
        SingleUpnlSig memory upnlSig //TODO: UpnlSig
    ) internal {
        MAStorage.Layout storage maLayout = MAStorage.layout();
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        require(
            !maLayout.partyBLiquidationStatus[msg.sender][origin],
            "PartyBFacet: PartyB isn't solvent"
        );
        require(
            !maLayout.partyBLiquidationStatus[msg.sender][recipient],
            "PartyBFacet: PartyB isn't solvent"
        );
        require(
            !MAStorage.layout().liquidationStatus[origin],
            "PartyBFacet: Origin isn't solvent"
        );
        require(
            !MAStorage.layout().liquidationStatus[recipient],
            "PartyBFacet: Recipient isn't solvent"
        );
        // deallocate from origin
        require(
            accountLayout.partyBAllocatedBalances[msg.sender][origin] >= amount,
            "PartyBFacet: Insufficient locked balance"
        );
        LibMuon.verifyPartyBUpnl(upnlSig, msg.sender, origin);
        int256 availableBalance = LibAccount.partyBAvailableForQuote(
            upnlSig.upnl,
            msg.sender,
            origin
        );
        require(availableBalance >= 0, "PartyBFacet: Available balance is lower than zero");
        require(uint256(availableBalance) >= amount, "PartyBFacet: Will be liquidatable");

        accountLayout.partyBNonces[msg.sender][origin] += 1;
        accountLayout.partyBAllocatedBalances[msg.sender][origin] -= amount;
        accountLayout.partyBAllocatedBalances[msg.sender][recipient] += amount;
    }

```

**Input Parameters**:&#x20;

* `amount`: the amount of tokens to deposit to the user.
* `origin`: the address of the user transferring collateral.
* `recipient`: the address of the recipient of the collateral.
* `upnlSig`: a cryptographic signature obtained from Muon to verify upnl.

**Verifications**:&#x20;

* Requires that the partyB is not marked as liquidated in regards to the origin address.
* Requires that partyB is not marked as liquidated in regards to the recipient address.
* Requires that both the origin and recipient are not marked as liquidated.
* Requires that the partyB allocated balance for the origin address exceeds the amount to transfer.
* Calls `LibMuon.verifyPartyBUpnl` to ensure that partyB will not be liquidated upon executing this function.

**Storage Interactions**: Adjusts allocated balances between users in `AccountStorage`.

**Events Emitted:**

```solidity
    event TransferAllocation(uint256 amount, address origin, address recipient);
```

### **allocateForPartyB()**

**Description**: Allocates funds to a `partyB`, with respect to a `partyA`.

```solidity
    function allocateForPartyB(
        uint256 amount,
        address partyA
    ) public whenNotPartyBActionsPaused notLiquidatedPartyB(msg.sender, partyA) onlyPartyB {
        AccountFacetImpl.allocateForPartyB(amount, partyA);
        emit AllocateForPartyB(msg.sender, partyA, amount);
    }
```

**Implementation:**

```solidity
function allocateForPartyB(uint256 amount, address partyA) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();

        require(accountLayout.balances[msg.sender] >= amount, "PartyBFacet: Insufficient balance");
        require(
            !MAStorage.layout().partyBLiquidationStatus[msg.sender][partyA],
            "PartyBFacet: PartyB isn't solvent"
        );
        accountLayout.balances[msg.sender] -= amount;
        accountLayout.partyBAllocatedBalances[msg.sender][partyA] += amount;
    }
```

**Input Parameters**:&#x20;

* `amount`: the amount of tokens to deposit to the user.
* `partyA`: the address of the partyA.

**Verifications**: Ensures Party B's is solvent to call this function.

**Storage Interactions**: Modifies balances in `AccountStorage` to reflect allocation.

**Events Emitted:**

```solidity
    event AllocateForPartyB(address partyB, address partyA, uint256 amount);
```

### **deallocateForPartyB()**

**Description**: Reverses allocations made for `partyB` with respect to a `partyA`, ensuring funds are returned to the user's balance.

```solidity
    function deallocateForPartyB(
        uint256 amount,
        address partyA,
        SingleUpnlSig memory upnlSig
    ) external whenNotPartyBActionsPaused notLiquidatedPartyB(msg.sender, partyA) notLiquidatedPartyA(partyA) onlyPartyB {
        AccountFacetImpl.deallocateForPartyB(amount, partyA, upnlSig);
        emit DeallocateForPartyB(msg.sender, partyA, amount);
    }
```

**Implementation:**

```solidity
function deallocateForPartyB(
        uint256 amount,
        address partyA,
        SingleUpnlSig memory upnlSig
    ) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        require(
            accountLayout.partyBAllocatedBalances[msg.sender][partyA] >= amount,
            "PartyBFacet: Insufficient allocated balance"
        );
        LibMuon.verifyPartyBUpnl(upnlSig, msg.sender, partyA);
        int256 availableBalance = LibAccount.partyBAvailableForQuote(
            upnlSig.upnl,
            msg.sender,
            partyA
        );
        require(availableBalance >= 0, "PartyBFacet: Available balance is lower than zero");
        require(uint256(availableBalance) >= amount, "PartyBFacet: Will be liquidatable");

        accountLayout.partyBNonces[msg.sender][partyA] += 1;
        accountLayout.partyBAllocatedBalances[msg.sender][partyA] -= amount;
        accountLayout.balances[msg.sender] += amount;
        accountLayout.withdrawCooldown[msg.sender] = block.timestamp;
    }
```

**Input Parameters**:&#x20;

* `amount`: the amount of tokens to deposit to the user.
* `partyA`: the address of the partyA.
* `upnlSig`: a cryptographic signature obtained from Muon to verify account upnl.

**Verifications**: Verifies the signature using `LibMuon.verifyPartyAUpnl`, then assesses if the user's `availableBalance` exceeds the amount they're requesting to deallocate.

**Storage Interactions**: Adjusts allocated and free balances, updates nonces and sets the cooldown start time in `AccountStorage`.\
