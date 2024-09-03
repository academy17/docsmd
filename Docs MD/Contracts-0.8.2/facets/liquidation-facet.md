# Liquidation Facet

This is the facet responsible for liquidations. Functions in these facets are called by liquidators, incentivized to earn liquidation fees from calling these functions in a timely manner.

In order to grasp the intricacies of the liquidation process, a fundamental understanding of the 'pending locked' concept is crucial. When a user sends a quote request, the corresponding amount of the position goes into a 'pending' state. During this phase, the user is restricted from opening other positions with that specific amount. Nonetheless, this amount continues to contribute to the user's allocated funds when assessing their liquidity status. Once Party B opens the position, this amount goes from the 'pending' to the 'locked' state.

## Liquidation Flow

### Liquidate Party A

For a better understanding of how a user gets liquidated, let’s look at one with a $1000 allocated balance as an example:

[![](https://github.com/SYMM-IO/protocol-core/raw/main/images/liq\_example1.png)](https://github.com/SYMM-IO/protocol-core/blob/main/images/liq\_example1.png)

User positions are all considered to be cross, meaning that in the above picture, values can be the sum of the equivalent values in 4 different positions.\
Pending locked values are from user quotes that have not been opened yet.

Now let’s say that the user is having a bad day, and one of their positions is sinking deep into loss:

[![](https://github.com/SYMM-IO/protocol-core/raw/main/images/liq\_example2.png)](https://github.com/SYMM-IO/protocol-core/blob/main/images/liq\_example2.png)

Each user position has a respective UPNL, which determines whether the position is in profit(positive UPNL) or loss(negative UPNL). Adding all those UPNLs, we get the user’s total UPNL. Now let’s see what happens if UPNL changes:

* Total upnl > 0: User is overall in profit
* \-500 < Total UPNL < 0: User’s locked MMs are supporting their positions
* \-700 < Total UPNL < -500: User’s free balance is now supporting their positions
* \-900 < Total UPNL < -700: User’s pending locked values are supporting their position
* Total UPNL < -900: User will be liquidated now

As this is a cross-system, whenever a user gets liquidated, all of their quotes and positions will go to a liquidated state, or in other words they all get canceled.

On the contract side, the liquidation of partyA is a four-step process. The liquidator should first liquidate the user:

```solidity
function liquidatePartyA(address partyA, SingleUpnlSig memory upnlSig);
```

At this point the user is marked as liquidated and the timestamp for that is recorded.

Then the liquidator should set the prices for all symbols associated with the user's positions:

```solidity
function setSymbolsPrice(address partyA, PriceSig memory priceSig);
// priceSig contains
// uint256[] symbolIds;
// uint256[] prices;
```

Then the liquidator should liquidate partyA pending positions:

```solidity
function liquidatePendingPositionsPartyA(address partyA);
```

And after that the liquidator should liquidate partyA open positions:

```solidity
function liquidatePositionsPartyA(address partyA, uint256[] memory quoteIds);
```

### Liquidate Party B

As alluded to in the 'Send Quote' section, Party B is required to allocate collateral prior to locking the quote, and this amount may need to be augmented depending on the current state of the opened position in the market. Consequently, if Party B's allocated collateral falls short due to a user realizing significant profits on one or multiple positions, Party B will undergo liquidation for that specific user. The system for Party B operates on a cross basis, albeit on a per-user framework. This implies that all of a user's positions with Party B are collectively considered in the calculations, while Party B's positions with other users won't influence the potential for liquidation in their relationship with the specific user.\
From the contract perspective, the liquidation of Party B is a two-stage process. The liquidator must initially liquidate Party B in relation to Party A. Following that, they must liquidate all positions that Party B holds with that specific Party A, which necessitates an additional contract call:

```solidity
function liquidatePartyB(address partyB, address partyA, SingleUpnlSig memory upnlSig);
```

```solidity
function liquidatePositionsPartyB(address partyB, address partyA, PriceSig memory priceSig);
```

## Liquidation Functions

### liquidatePartyA()

**Description**: Initiates the liquidation process for Party A if they are insolvent, verifying their financial status using a liquidation signature and updating system records to reflect the liquidation status.

```solidity
    function liquidatePartyA(address partyA, LiquidationSig memory liquidationSig) internal {
        MAStorage.Layout storage maLayout = MAStorage.layout();

        LibMuon.verifyLiquidationSig(liquidationSig, partyA);
        require(
            block.timestamp <= liquidationSig.timestamp + MuonStorage.layout().upnlValidTime,
            "LiquidationFacet: Expired signature"
        );
        int256 availableBalance = LibAccount.partyAAvailableBalanceForLiquidation(
            liquidationSig.upnl,
            partyA
        );
        require(availableBalance < 0, "LiquidationFacet: PartyA is solvent");
        maLayout.liquidationStatus[partyA] = true;
        AccountStorage.layout().liquidationDetails[partyA] = LiquidationDetail({
            liquidationId: liquidationSig.liquidationId,
            liquidationType: LiquidationType.NONE,
            upnl: liquidationSig.upnl,
            totalUnrealizedLoss: liquidationSig.totalUnrealizedLoss,
            deficit: 0,
            liquidationFee: 0,
            timestamp: liquidationSig.timestamp,
            involvedPartyBCounts: 0,
            partyAAccumulatedUpnl: 0,
            disputed: false
        });
        AccountStorage.layout().liquidators[partyA].push(msg.sender);
    }

```

**Parameters**:

* **`partyA`**: The address of Party A being liquidated.
* **`liquidationSig`**: A muon signature containing the details and signatures necessary for validating the liquidation.

**Functionality**:

**Signature Verification**:

* Uses `LibMuon.verifyLiquidationSig` to authenticate the liquidation request based on the provided signature.

**Validation**:

* Ensures that the liquidation signature is still valid within the time constraints set by `MuonStorage.layout().upnlValidTime`.
* Checks the solvency of Party A using `LibAccount.partyAAvailableBalanceForLiquidation`. If the available balance is negative, it confirms insolvency.

**Record Updates**:

* Sets the liquidation status for Party A to true in `MAStorage`.
* Records the details of the liquidation in `AccountStorage`, including the timestamp and specifics from the `liquidationSig`.

**Liquidator Tracking**:

* Logs the address of the entity initiating the liquidation in Party A's liquidator list.

### setSymbolsPrice()

**Description**: Updates the prices of symbols associated with Party A's account during the liquidation process, as specified in the liquidation signature, and potentially adjusts the type of liquidation.

```solidity
    function setSymbolsPrice(address partyA, LiquidationSig memory liquidationSig) internal {
        MAStorage.Layout storage maLayout = MAStorage.layout();
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();

        LibMuon.verifyLiquidationSig(liquidationSig, partyA);
        require(maLayout.liquidationStatus[partyA], "LiquidationFacet: PartyA is solvent");
        require(
            keccak256(accountLayout.liquidationDetails[partyA].liquidationId) ==
            keccak256(liquidationSig.liquidationId),
            "LiquidationFacet: Invalid liqiudationId"
        );
        for (uint256 index = 0; index < liquidationSig.symbolIds.length; index++) {
            accountLayout.symbolsPrices[partyA][liquidationSig.symbolIds[index]] = Price(
                liquidationSig.prices[index],
                accountLayout.liquidationDetails[partyA].timestamp
            );
        }

        int256 availableBalance = LibAccount.partyAAvailableBalanceForLiquidation(
            liquidationSig.upnl,
            partyA
        );
        if (accountLayout.liquidationDetails[partyA].liquidationType == LiquidationType.NONE) {
            if (uint256(- availableBalance) < accountLayout.lockedBalances[partyA].lf) {
                uint256 remainingLf = accountLayout.lockedBalances[partyA].lf -
                                    uint256(- availableBalance);
                accountLayout.liquidationDetails[partyA].liquidationType = LiquidationType.NORMAL;
                accountLayout.liquidationDetails[partyA].liquidationFee = remainingLf;
            } else if (
                uint256(- availableBalance) <=
                accountLayout.lockedBalances[partyA].lf + accountLayout.lockedBalances[partyA].cva
            ) {
                uint256 deficit = uint256(- availableBalance) -
                                        accountLayout.lockedBalances[partyA].lf;
                accountLayout.liquidationDetails[partyA].liquidationType = LiquidationType.LATE;
                accountLayout.liquidationDetails[partyA].deficit = deficit;
            } else {
                uint256 deficit = uint256(- availableBalance) -
                                        accountLayout.lockedBalances[partyA].lf -
                                        accountLayout.lockedBalances[partyA].cva;
                accountLayout.liquidationDetails[partyA].liquidationType = LiquidationType.OVERDUE;
                accountLayout.liquidationDetails[partyA].deficit = deficit;
            }
            AccountStorage.layout().liquidators[partyA].push(msg.sender);
        }
    }

```

**Parameters**:

* **`partyA`**: The address of Party A undergoing liquidation.
* **`liquidationSig`**: The liquidation signature data that includes the prices of symbols to be updated.

**Functionality**:

**Validation**:

* Verifies that Party A is currently under liquidation.
* Confirms that the liquidation ID in the signature matches the one on record, ensuring consistency and validity in the liquidation process.

**Price Updates**:

* Updates the recorded prices for the specified symbols in Party A's account to reflect current market values as stated in the `liquidationSig`.

**Financial Assessment**:

* Re-evaluates Party A's solvency post-price update to determine the appropriate type of liquidation based on the updated financial data. Depending on the severity of the financial shortfall, adjusts the liquidation details to reflect whether it is a normal, late, or overdue liquidation. This impacts the fee amount the liquidator will receive.

**Liquidator Tracking**:

Adjusts the liquidation details, including potential fees and deficits, based on the re-evaluated solvency condition and adds the liquidator to Party A's list of liquidators if they are not already listed.

### liquidatePendingPositionsPartyA()

**Description**: Executes the liquidation of all pending quotes for Party A when they are deemed insolvent.

```solidity
    function liquidatePendingPositionsPartyA(address partyA) internal {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        require(
            MAStorage.layout().liquidationStatus[partyA],
            "LiquidationFacet: PartyA is solvent"
        );
        for (uint256 index = 0; index < quoteLayout.partyAPendingQuotes[partyA].length; index++) {
            Quote storage quote = quoteLayout.quotes[
                                    quoteLayout.partyAPendingQuotes[partyA][index]
                ];
            if (
                (quote.quoteStatus == QuoteStatus.LOCKED ||
                    quote.quoteStatus == QuoteStatus.CANCEL_PENDING) &&
                quoteLayout.partyBPendingQuotes[quote.partyB][partyA].length > 0
            ) {
                delete quoteLayout.partyBPendingQuotes[quote.partyB][partyA];
                AccountStorage
                .layout()
                .partyBPendingLockedBalances[quote.partyB][partyA].makeZero();
            }
            AccountStorage.layout().partyAReimbursement[partyA] += LibQuote.getTradingFee(quote.id);
            quote.quoteStatus = QuoteStatus.CANCELED;
            quote.statusModifyTimestamp = block.timestamp;
        }
        AccountStorage.layout().pendingLockedBalances[partyA].makeZero();
        delete quoteLayout.partyAPendingQuotes[partyA];
    }

```

**Parameters**:

* **`partyA`**: The address of Party A whose pending positions are to be liquidated.

**Functionality**:

**Validation**:

* Confirms that Party A is marked as insolvent in `MAStorage`.

**Liquidation Process**:

* Iterates over all pending quotes of Party A, checking each quote for specific conditions such as being locked or cancel-pending.
* For each quote that meets the criteria:
  * Clears any pending quotes linked to Party B to prevent future claims or actions.
  * Resets Party B’s pending locked balances associated with Party A, essentially zeroing out any remaining commitments.
  * Accumulates any potential trading fees from the cancelled quotes to a reimbursement account for Party A.

**Final Adjustments**:

* Marks all processed quotes as `CANCELED` and updates their status modification timestamps to the current block timestamp.
* Clears all of Party A's pending locked balances to reflect their new balances post-liquidation.
* Removes all records of Party A's pending quotes from `QuoteStorage`.

**Storage Interactions**:

Modifies `QuoteStorage` and `AccountStorage` to update the status of quotes, and adjusts the account balances.

### liquidatePositionsPartyA()

**Description**: Conducts the systematic liquidation of specific trading positions associated with Party A, who has been declared insolvent, ensuring that each position is evaluated and processed according to the current market conditions and predefined liquidation rules.

```solidity
    function liquidatePositionsPartyA(
    address partyA, 
    uint256[] memory quoteIds
    ) internal returns (bool){
```

**Parameters**:

* **`partyA`**: The address of Party A, whose positions are being liquidated.
* **`quoteIds`**: An array of quoteIds that specify which positions of Party A are to be liquidated.

**Returns**:

* **`bool`**: Returns `true` if there is a discrepancy in the final UPnL calculations post-liquidation, indicating a dispute; otherwise, `false`.

**Functionality**:

**Validation**:

* Confirms that Party A is officially in a state of liquidation.
* Ensures each quote is in a state eligible for liquidation (either `OPENED`, `CLOSE_PENDING`, or `CANCEL_CLOSE_PENDING`).
* Checks that none of the counterparties (Party B) associated with these quotes are currently undergoing their own liquidation process.

**Liquidation Process**:

* Iterates through each quote ID provided:
  * Changes the quote's status to `LIQUIDATED` and records the liquidation time.
  * Evaluates the financial outcome of the liquidation using current market prices, adjusting Party A's and Party B's accounts accordingly.
  * Manages settlement states between Party A and Party B, updating CVAs and calculating actual and expected amounts based on the type of liquidation (Normal, Late, or Overdue).

**Settlement and Reconciliation**:

* Adjusts locked balances and accounts for trading results.
* Updates liquidation details based on outcomes, including accumulated unrealized profit or loss.
* Tracks and decrements the count of Party A’s open positions and ensures all adjustments are accounted for each Party B involved.
* Resolves the final settlement amounts between Party A and each Party B, applying these to the overall liquidation calculations.

**Dispute Detection**:

* At the end of the process, checks if the accumulated unrealized profit or loss matches the expected outcome. If discrepancies exist, flags the liquidation as disputed.

**Storage Interactions**:

* Uses `AccountStorage` to update financial states and balances.
* Manipulates `QuoteStorage` to alter quote statuses and manage associated data.
* Adjusts `MAStorage` for tracking overall liquidation status and details.

### resolveLiquidationDispute()

**Description**: This function is used to finalize the outcomes of liquidation disputes by directly adjusting the recorded settlement amounts between Party A and various Party Bs, based on agreed figures.

```solidity
    function resolveLiquidationDispute(
        address partyA,
        address[] memory partyBs,
        int256[] memory amounts,
        bool disputed
    ) internal {
        AccountStorage.layout().liquidationDetails[partyA].disputed = disputed;
        require(partyBs.length == amounts.length, "LiquidationFacet: Invalid length");
        for (uint256 i = 0; i < partyBs.length; i++) {
            AccountStorage.layout().settlementStates[partyA][partyBs[i]].actualAmount = amounts[i];
        }
    }

```

**Parameters**:

* **`partyA`**: The address of Party A, whose liquidation dispute is being resolved.
* **`partyBs`**: An array of addresses for Party Bs involved in the liquidation with Party A.
* **`amounts`**: An array of amounts corresponding to the final settled amounts for each Party B.
* **`disputed`**: A boolean flag indicating the resolution status of the dispute; `true` if the dispute remains unresolved after adjustments, otherwise `false`.

**Functionality**:

**Dispute Status Update**:

* Directly updates the dispute status for Party A’s liquidation case to reflect whether the dispute is ongoing or resolved.

**Validation**:

* Checks that the lengths of the `partyBs` and `amounts` arrays are equal, ensuring each Party B has a corresponding settlement amount.

**Adjustment of Settlement Amounts**:

Iterates through each Party B provided in the `partyBs` array and sets the `actualAmount` in the settlement state for each Party B to the respective value provided in the `amounts` array.&#x20;

**Storage Interactions**:

Modifies `AccountStorage` to update dispute statuses and settlement amounts related to the liquidation details of Party A.

### settlePartyALiquidation()

**Description**: This function concludes the liquidation process for Party A by settling all outstanding balances with various Party Bs, ensuring all financial obligations are met and clearing Party A's related records.

```solidity
    function settlePartyALiquidation(address partyA, address[] memory partyBs) internal returns (int256[] memory settleAmounts){
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        require(
            quoteLayout.partyAPositionsCount[partyA] == 0 &&
            quoteLayout.partyAPendingQuotes[partyA].length == 0,
            "LiquidationFacet: PartyA has still open positions"
        );
        require(
            MAStorage.layout().liquidationStatus[partyA],
            "LiquidationFacet: PartyA is solvent"
        );
        require(
            !accountLayout.liquidationDetails[partyA].disputed,
            "LiquidationFacet: PartyA liquidation process get disputed"
        );
        settleAmounts = new int256[](partyBs.length);
        for (uint256 i = 0; i < partyBs.length; i++) {
            address partyB = partyBs[i];
            require(
                accountLayout.settlementStates[partyA][partyB].pending,
                "LiquidationFacet: PartyB is not in settlement"
            );
            accountLayout.settlementStates[partyA][partyB].pending = false;
            accountLayout.liquidationDetails[partyA].involvedPartyBCounts -= 1;

            int256 settleAmount = accountLayout.settlementStates[partyA][partyB].actualAmount;
            accountLayout.partyBAllocatedBalances[partyB][partyA] += accountLayout
                .settlementStates[partyA][partyB].cva;
            if (settleAmount < 0) {
                accountLayout.partyBAllocatedBalances[partyB][partyA] += uint256(- settleAmount);
                settleAmounts[i] = settleAmount;
            } else {
                if (
                    accountLayout.partyBAllocatedBalances[partyB][partyA] >= uint256(settleAmount)
                ) {
                    accountLayout.partyBAllocatedBalances[partyB][partyA] -= uint256(settleAmount);
                    settleAmounts[i] = settleAmount;
                } else {
                    settleAmounts[i] = int256(accountLayout.partyBAllocatedBalances[partyB][partyA]);
                    accountLayout.partyBAllocatedBalances[partyB][partyA] = 0;
                }
            }
            delete accountLayout.settlementStates[partyA][partyB];
        }
        if (accountLayout.liquidationDetails[partyA].involvedPartyBCounts == 0) {
            accountLayout.allocatedBalances[partyA] = accountLayout.partyAReimbursement[partyA];
            accountLayout.partyAReimbursement[partyA] = 0;
            accountLayout.lockedBalances[partyA].makeZero();

            uint256 lf = accountLayout.liquidationDetails[partyA].liquidationFee;
            if (lf > 0) {
                accountLayout.allocatedBalances[accountLayout.liquidators[partyA][0]] += lf / 2;
                accountLayout.allocatedBalances[accountLayout.liquidators[partyA][1]] += lf / 2;
            }
            delete accountLayout.liquidators[partyA];
            delete accountLayout.liquidationDetails[partyA].liquidationType;
            MAStorage.layout().liquidationStatus[partyA] = false;
            accountLayout.partyANonces[partyA] += 1;
        }
    }

```

**Parameters**:

* **`partyA`**: The address of Party A whose liquidation is being settled.
* **`partyBs`**: An array of Party B addresses involved in the settlements.

**Returns**:

* **`settledAmounts(int256[])`:** Array of settlement amounts for each Party B, reflecting the final settled amounts either owed to or from Party A.

**Functionality**:

**Preconditions**:

* Ensures that Party A has no open or pending quotes remaining, confirming all trading positions are closed.
* Verifies that Party A is officially in a state of liquidation.
* Checks that there is no ongoing dispute regarding Party A’s liquidation to prevent settlement under unresolved claims.

**Settlement Initialization**:

* Initializes an array to store the settlement amounts for each Party B.

**Settlement Process**:

* Iterates through each Party B:
  * Confirms that there is a pending settlement state for each Party B.
  * Marks the settlement as completed by setting `pending` to false and decrements the count of involved Party Bs.
  * Calculates the settle amount based on the `actualAmount` recorded in the settlement states.
  * Adjusts Party B’s allocated balances by adding any credit value adjustment (CVA) and updating balances based on the settle amount.
  * Clears the settlement state for Party B once processed.

**Post-Settlement** :

* After all Party Bs have been settled, checks if all involved Party Bs have been processed.
* Adjusts Party A's balances to reflect any reimbursements due.
* Resets Party A's locked balances to zero and redistributes any applicable liquidation fees among the liquidators.
* Clears all liquidation-related records for Party A and marks the liquidation as resolved by setting the liquidation status to false.

**Storage Interactions**:

* **`AccountStorage`**: Updates financial balances, clears settlement states, and manages liquidation records.
* **`QuoteStorage`**: Ensures that no open or pending positions remain for Party A.
* **`MAStorage`**: Updates the overall liquidation status of Party A.

### liquidatePartyB()

**Description**: Executes the liquidation of Party B who is financially insolvent in relation to Party A, adjusting their balances, settling pending quotes, and distributing any remaining assets according to predefined rules.

```solidity
    function liquidatePartyB(
        address partyB,
        address partyA,
        SingleUpnlSig memory upnlSig
    ) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();

        LibMuon.verifyPartyBUpnl(upnlSig, partyB, partyA);
        int256 availableBalance = LibAccount.partyBAvailableBalanceForLiquidation(
            upnlSig.upnl,
            partyB,
            partyA
        );

        require(availableBalance < 0, "LiquidationFacet: partyB is solvent");
        uint256 liquidatorShare;
        uint256 remainingLf;
        if (uint256(- availableBalance) < accountLayout.partyBLockedBalances[partyB][partyA].lf) {
            remainingLf =
                accountLayout.partyBLockedBalances[partyB][partyA].lf -
                uint256(- availableBalance);
            liquidatorShare = (remainingLf * maLayout.liquidatorShare) / 1e18;

            maLayout.partyBPositionLiquidatorsShare[partyB][partyA] =
                (remainingLf - liquidatorShare) /
                quoteLayout.partyBPositionsCount[partyB][partyA];
        } else {
            maLayout.partyBPositionLiquidatorsShare[partyB][partyA] = 0;
        }

        maLayout.partyBLiquidationStatus[partyB][partyA] = true;
        maLayout.partyBLiquidationTimestamp[partyB][partyA] = upnlSig.timestamp;

        uint256[] storage pendingQuotes = quoteLayout.partyAPendingQuotes[partyA];

        for (uint256 index = 0; index < pendingQuotes.length;) {
            Quote storage quote = quoteLayout.quotes[pendingQuotes[index]];
            if (
                quote.partyB == partyB &&
                (quote.quoteStatus == QuoteStatus.LOCKED ||
                    quote.quoteStatus == QuoteStatus.CANCEL_PENDING)
            ) {
                accountLayout.pendingLockedBalances[partyA].subQuote(quote);
                accountLayout.allocatedBalances[partyA] += LibQuote.getTradingFee(quote.id);
                pendingQuotes[index] = pendingQuotes[pendingQuotes.length - 1];
                pendingQuotes.pop();
                quote.quoteStatus = QuoteStatus.CANCELED;
                quote.statusModifyTimestamp = block.timestamp;
            } else {
                index++;
            }
        }
        accountLayout.allocatedBalances[partyA] +=
            accountLayout.partyBAllocatedBalances[partyB][partyA] -
            remainingLf;

        delete quoteLayout.partyBPendingQuotes[partyB][partyA];
        accountLayout.partyBAllocatedBalances[partyB][partyA] = 0;
        accountLayout.partyBLockedBalances[partyB][partyA].makeZero();
        accountLayout.partyBPendingLockedBalances[partyB][partyA].makeZero();
        accountLayout.partyANonces[partyA] += 1;

        if (liquidatorShare > 0) {
            accountLayout.allocatedBalances[msg.sender] += liquidatorShare;
        }
    }

```

**Parameters**:

* **`partyB`**: The address of Party B being liquidated.
* **`partyA`**: The address of Party A associated with the transactions and balances of Party B.
* **`upnlSig`**: Muon Signature containing the unrealized profit and loss (UPnL) data for Party B,.

**Functionality**:

**Validation and Checks**:

* Verifies the UPnL signature using `LibMuon.verifyPartyBUpnl`.
* Checks Party B's available balance for liquidation; liquidation proceeds only if this balance is negative, indicating insolvency.

**Liquidation Computations**:

* Calculates the liquidation fee for the liquidator based on the remaining liquidator fee (`lf`) after covering the negative balance.
* Determines the proportion of the remaining `lf` to be distributed among the liquidators based on the count of Party B's positions associated with Party A.

**Update Liquidation Status**:

* Marks Party B's liquidation status as true in `MAStorage` and records the timestamp of the liquidation.

**Resolve Pending Quotes**:

* Iterates through all pending quotes of Party A linked to Party B:
  * Cancels each quote if Party B is the counterpart and the quote is either locked or cancel-pending.
  * Removes the quote from Party A's pending list and adjusts the financial records accordingly, including reimbursing any trading fees to Party A.

**Financial Adjustments**:

* Adjusts Party A’s balances to include any allocated balances of Party B minus the liquidation share given to the liquidators.
* Clears all locked and pending locked balances for Party B related to Party A.

**Distribution of Liquidation Proceeds**:

* If a liquidator share is calculated, it is added to the liquidator's allocated balances, compensating them for their role in managing the liquidation process.

**Storage Interactions**:

* Modifies multiple storage structures including `AccountStorage`, `MAStorage`, and `QuoteStorage`.

### liquidatePositionsPartyB()

**Description**: Processes the liquidation of all relevant trading positions associated with Party B, with respect to Party A.

```solidity
    function liquidatePositionsPartyB(
        address partyB,
        address partyA,
        QuotePriceSig memory priceSig
    ) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();

        LibMuon.verifyQuotePrices(priceSig);
        require(
            priceSig.timestamp <=
            maLayout.partyBLiquidationTimestamp[partyB][partyA] + maLayout.liquidationTimeout,
            "LiquidationFacet: Invalid signature"
        );
        require(
            maLayout.partyBLiquidationStatus[partyB][partyA],
            "LiquidationFacet: PartyB is solvent"
        );
        require(
            maLayout.partyBLiquidationTimestamp[partyB][partyA] <= priceSig.timestamp,
            "LiquidationFacet: Expired signature"
        );
        for (uint256 index = 0; index < priceSig.quoteIds.length; index++) {
            Quote storage quote = quoteLayout.quotes[priceSig.quoteIds[index]];
            require(
                quote.quoteStatus == QuoteStatus.OPENED ||
                quote.quoteStatus == QuoteStatus.CLOSE_PENDING ||
                quote.quoteStatus == QuoteStatus.CANCEL_CLOSE_PENDING,
                "LiquidationFacet: Invalid state"
            );
            require(
                quote.partyA == partyA && quote.partyB == partyB,
                "LiquidationFacet: Invalid party"
            );

            quote.quoteStatus = QuoteStatus.LIQUIDATED;
            quote.statusModifyTimestamp = block.timestamp;

            accountLayout.lockedBalances[partyA].subQuote(quote);

            quote.avgClosedPrice =
                (quote.avgClosedPrice *
                quote.closedAmount +
                    LibQuote.quoteOpenAmount(quote) *
                    priceSig.prices[index]) /
                (quote.closedAmount + LibQuote.quoteOpenAmount(quote));
            quote.closedAmount = quote.quantity;

            LibQuote.removeFromOpenPositions(quote.id);
            quoteLayout.partyAPositionsCount[partyA] -= 1;
            quoteLayout.partyBPositionsCount[partyB][partyA] -= 1;
        }
        if (maLayout.partyBPositionLiquidatorsShare[partyB][partyA] > 0) {
            accountLayout.allocatedBalances[msg.sender] +=
                maLayout.partyBPositionLiquidatorsShare[partyB][partyA] *
                priceSig.quoteIds.length;
        }

        if (quoteLayout.partyBPositionsCount[partyB][partyA] == 0) {
            maLayout.partyBLiquidationStatus[partyB][partyA] = false;
            maLayout.partyBLiquidationTimestamp[partyB][partyA] = 0;
            accountLayout.partyBNonces[partyB][partyA] += 1;
        }
    }

```

**Parameters**:

* **`partyB`**: The address of Party B, whose positions are being liquidated.
* **`partyA`**: The address of Party A associated with Party B's positions.
* **`priceSig`**: A price signature containing the prices for the quotes being liquidated, along with their identifiers and a timestamp.

```solidity
struct QuotePriceSig {
    bytes reqId;
    uint256 timestamp;
    uint256[] quoteIds;
    uint256[] prices;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

**Functionality**:

**Price Verification**:

* Utilizes `LibMuon.verifyQuotePrices` to ensure the prices provided in `priceSig` are accurate.

**Preconditions and Checks**:

* Confirms the price signature is within the liquidation timeout period relative to Party B's last recorded liquidation timestamp.
* Ensures Party B is indeed marked as insolvent, permitting liquidation.
* Validates the timing of the signature against the liquidation timestamp.

**Liquidation of Positions**:

* Iterates through each quote ID provided in the `priceSig`:
  * Validates the current state of each quote, ensuring it is either opened, close pending, or cancel close pending.
  * Confirms the parties involved in the quote match Party A and Party B.
  * Updates the quote status to `LIQUIDATED` and sets the timestamp for this status change.
  * Adjusts the closed price average based on the new prices provided, recalculating based on the total quantities involved.
  * Marks the quote as fully closed and updates various counters and records to reflect this change.

**Distribution of Liquidation Shares**:

* If there is a liquidator’s share defined for the positions being liquidated, calculates the total compensation for the liquidator based on the number of quotes processed.

**Final Adjustments**:

* Checks if all positions for Party B against Party A are liquidated:
  * If so, resets the liquidation status and related timestamps for Party B, effectively clearing their liquidation state.
  * Increments nonce for Party B and Party A to ensure future transactions are processed based on the updated state.

**Storage Interactions**:

* Modifies `AccountStorage` to adjust financial balances and update transactional records.
* Updates `QuoteStorage` to change the statuses of the quotes and manage position counts.
* Adjusts `MAStorage` to update liquidation records and statuses.
