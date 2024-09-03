# MultiAccount

In order to place trades using a SYMM frontend solution or otherwise, it's necessary to create a sub-account using a `MultiAccount` contract that is whitelisted to interact with the hedger. Users are required to provide a name as an input parameter when they create a sub-account. All positions on a sub-account are in **CROSS**, but positions between sub-accounts are **ISOLATED**.

## MultiAccount Contract Functions

## **Initialization and Setup**

### **initialize()**

Initializes the MultiAccount contract with roles and addresses necessary for operation.

```solidity
    function initialize(address admin, address symmioAddress_, bytes memory accountImplementation_) public initializer {
        __Pausable_init();
        __AccessControl_init();

        _grantRole(DEFAULT_ADMIN_ROLE, admin);
        _grantRole(PAUSER_ROLE, admin);
        _grantRole(UNPAUSER_ROLE, admin);
        _grantRole(SETTER_ROLE, admin);
        accountsAdmin = admin;
        symmioAddress = symmioAddress_;
        accountImplementation = accountImplementation_;
    }

```

* **Parameters**:
  * `admin`: The administrator's address who will have default and control roles.
  * `symmioAddress_`: Address of the Symmio platform to interact with.
  * `accountImplementation_`: Bytecode for the account implementation to be used (SymmioPartyA).

## **Account Management**

### **addAccount()**

Creates a new sub-account for a user with a specified name.

```solidity
    function addAccount(string memory name) external whenNotPaused {
        address account = _deployPartyA();
        indexOfAccount[account] = accounts[msg.sender].length;
        accounts[msg.sender].push(Account(account, name));
        owners[account] = msg.sender;
        emit AddAccount(msg.sender, account, name);
    }
```

**Parameters**:

* `name`: Name of the sub-account.

**Emits**: `AddAccount` event upon successful creation.

### **editAccountName()**

Allows the owner to change the name of an existing sub-account.

```solidity
    function editAccountName(address accountAddress, string memory name) external whenNotPaused {
        uint256 index = indexOfAccount[accountAddress];
        accounts[msg.sender][index].name = name;
        emit EditAccountName(msg.sender, accountAddress, name);
    }
```

**Parameters**:

* `accountAddress`: Address of the account to rename.
* `name`: New name for the account.

**Emits**: `EditAccountName` event.

### **depositForAccount()**

Deposits funds into a sub-account from the owner's balance.

```solidity
    function depositForAccount(address account, uint256 amount) external onlyOwner(account, msg.sender) whenNotPaused {
        address collateral = ISymmio(symmioAddress).getCollateral();
        IERC20Upgradeable(collateral).safeTransferFrom(msg.sender, address(this), amount);
        IERC20Upgradeable(collateral).safeApprove(symmioAddress, amount);
        ISymmio(symmioAddress).depositFor(account, amount);
        emit DepositForAccount(msg.sender, account, amount);
    }
```

**Parameters**:

* `account`: Address of the sub-account.
* `amount`: Amount to deposit.

**Emits**: `DepositForAccount` event.

### **depositAndAllocateForAccount()**

Deposits funds into a sub-account and allocates them for trading.

```solidity
    function depositAndAllocateForAccount(address account, uint256 amount) external onlyOwner(account, msg.sender) whenNotPaused {
        address collateral = ISymmio(symmioAddress).getCollateral();
        IERC20Upgradeable(collateral).safeTransferFrom(msg.sender, address(this), amount);
        IERC20Upgradeable(collateral).safeApprove(symmioAddress, amount);
        ISymmio(symmioAddress).depositFor(account, amount);
        uint256 amountWith18Decimals = (amount * 1e18) / (10 ** IERC20Metadata(collateral).decimals());
        bytes memory _callData = abi.encodeWithSignature("allocate(uint256)", amountWith18Decimals);
        innerCall(account, _callData);
        emit DepositForAccount(msg.sender, account, amount);
        emit AllocateForAccount(msg.sender, account, amountWith18Decimals);
    }
```

**Parameters**:

* `account`: Address of the sub-account.
* `amount`: Amount to deposit and allocate.

**Emits**: `DepositForAccount` and `AllocateForAccount` events.

### **withdrawFromAccount()**

Withdraws funds from a sub-account back to the owner's address.

```solidity
    function withdrawFromAccount(address account, uint256 amount) external onlyOwner(account, msg.sender) whenNotPaused {
        bytes memory _callData = abi.encodeWithSignature("withdrawTo(address,uint256)", owners[account], amount);
        emit WithdrawFromAccount(msg.sender, account, amount);
        innerCall(account, _callData);
    }
```

**Parameters**:

* `account`: Address of the sub-account.
* `amount`: Amount to withdraw.

**Emits**: `WithdrawFromAccount` event.

## **Access Control**

### **delegateAccess()**

Allows the owner of a sub-account to delegate control over specific functions to another address.

```solidity
    function delegateAccess(address account, address target, bytes4 selector, bool state) external onlyOwner(account, msg.sender) {
        require(target != msg.sender && target != account, "MultiAccount: invalid target");
        emit DelegateAccess(account, target, selector, state);
        delegatedAccesses[account][target][selector] = state;
    }
```

**Parameters**:

* `account`: Address of the sub-account.
* `target`: Address to which access is delegated.
* `selector`: Function selector for which access is granted.
* `state`: Boolean to enable or disable access. `true` sets the delegation state to enabled, allowing the delegate to call the specified function on behalf of the sub-account

**Emits**: `DelegateAccess` event.

### **delegateAccesses()**

Batch version of `delegateAccess` for multiple function selectors.

```solidity
    function delegateAccesses(address account, address target, bytes4[] memory selector, bool state) external onlyOwner(account, msg.sender) {
        require(target != msg.sender && target != account, "MultiAccount: invalid target");
        for (uint256 i = selector.length; i != 0; i--) {
            delegatedAccesses[account][target][selector[i - 1]] = state;
            emit DelegateAccess(account, target, selector[i - 1], state);
        }
    }
```

**Parameters**:

* `account`, `target`, `selector[]`, and `state` as in `delegateAccess`.

**Emits**: `DelegateAccess` event for each selector.

## **Configuration and Address Management**

### **setAccountImplementation()**

Sets new account implementation bytecode.

```solidity
    function setAccountImplementation(bytes memory accountImplementation_) external onlyRole(SETTER_ROLE) {
        emit SetAccountImplementation(accountImplementation, accountImplementation_);
        accountImplementation = accountImplementation_;
    }
```

**Parameters**:

* `accountImplementation_`: New bytecode for account implementation.

**Emits**: `SetAccountImplementation` event.

### **setSymmioAddress()**

Updates the address of the Symmio platform.

```solidity
    function setSymmioAddress(address addr) external onlyRole(SETTER_ROLE) {
        emit SetSymmioAddress(symmioAddress, addr);
        symmioAddress = addr;
    }
```

**Parameters**:

`addr`: New address for the Symmio platform.

**Emits**: `SetSymmioAddress` event.

## **Pausable Functionality**

### **pause()**

Pauses all pausable actions in the contract, preventing execution. Only callable by addresses with the `PAUSER_ROLE`.

```solidity
    function pause() external onlyRole(PAUSER_ROLE) {
        _pause();
    }
```

### **unpause()**

Resumes all actions in the contract after being paused. Only callable by addresses with the `UNPAUSER_ROLE`.

```solidity
    function unpause() external onlyRole(UNPAUSER_ROLE) {
        _unpause();
    }
```

## **Internal and Utility Functions**

### **\_deployPartyA()**

Deploys a new sub-account using the current account implementation.

```solidity
    function _deployPartyA() internal returns (address account) {
        bytes32 salt = keccak256(abi.encodePacked("MultiAccount_", saltCounter));
        saltCounter += 1;

        bytes memory bytecode = abi.encodePacked(accountImplementation, abi.encode(accountsAdmin, address(this), symmioAddress));
        account = _deployContract(bytecode, salt);
        return account;
    }
```

**Returns**: Address of the newly deployed account.

### **\_deployContract()**

Internal function to deploy contracts using the create2 opcode.

```solidity
    function _deployContract(bytes memory bytecode, bytes32 salt) internal returns (address contractAddress) {
        assembly {
            contractAddress := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        require(contractAddress != address(0), "MultiAccount: create2 failed");
        emit DeployContract(msg.sender, contractAddress);
        return contractAddress;
    }
```

**Parameters**:

* `bytecode`: Compiled bytecode of the contract.
* `salt`: Salt used for create2 to determine the address.

**Returns**: Address of the deployed contract.

### **\_call()**

Internal function to invoke methods on other contracts.

```solidity
    function _call(address account, bytes[] memory _callDatas) public whenNotPaused {
        bool isOwner = owners[account] == msg.sender;
        for (uint8 i; i < _callDatas.length; i++) {
            bytes memory _callData = _callDatas[i];
            if (!isOwner) {
                require(_callData.length >= 4, "MultiAccount: Invalid call data");
                bytes4 functionSelector;
                assembly {
                    functionSelector := mload(add(_callData, 0x20))
                }
                require(delegatedAccesses[account][msg.sender][functionSelector], "MultiAccount: Unauthorized access");
            }
            innerCall(account, _callData);
        }
    }
```

**Parameters**:

* `account`: Account from which the call is made.
* `_callDatas[]`: Array of call data to be executed.

Executes multiple calls in a single transaction if authorized.

## View Functions

### **getAccountsLength()**

Returns the number of accounts owned by a user.

```solidity
    function getAccountsLength(address user) external view returns (uint256) {
        return accounts[user].length;
    }
```

**Parameters**:

* `user`: Address of the user.

**Returns**: Number of accounts.

### **getAccounts()**

Retrieves a list of sub-accounts owned by a user.

```solidity
    function getAccounts(address user, uint256 start, uint256 size) external view returns (Account[] memory) {
        uint256 len = size > accounts[user].length - start ? accounts[user].length - start : size;
        Account[] memory userAccounts = new Account[](len);
        for (uint256 i = start; i < start + len; i++) {
            userAccounts[i - start] = accounts[user][i];
        }
        return userAccounts;
    }
```

**Parameters**:

* `user`: Owner's address.
* `start`: Start index for pagination.
* `size`: Number of accounts to return.

**Returns**: Array of `Account` structs.
