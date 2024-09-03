# PartyB Facet

The PartyB Facet contains functions related to accepting positions quotes sent by partyA.

## lockQuote()

Once a user issues a quote, any PartyB can secure it by providing sufficient funds, based on their estimated profit and loss from opening the position. This is referred to as a 'lock quote' as it bars other PartyBs from interacting with the quote. The process of reserving funds is accomplished through the subsequent contract methods:

```solidity
    function lockQuote(uint256 quoteId, SingleUpnlSig memory upnlSig, bool increaseNonce) internal {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();

        Quote storage quote = quoteLayout.quotes[quoteId];
        LibMuon.verifyPartyBUpnl(upnlSig, msg.sender, quote.partyA);
        LibPartyB.checkPartyBValidationToLockQuote(quoteId, upnlSig.upnl);
        if (increaseNonce) {
            accountLayout.partyBNonces[msg.sender][quote.partyA] += 1;
        }
        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.LOCKED;
        quote.partyB = msg.sender;
        // lock funds for partyB
        accountLayout.partyBPendingLockedBalances[msg.sender][quote.partyA].addQuote(quote);
        quoteLayout.partyBPendingQuotes[msg.sender][quote.partyA].push(quote.id);
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote that Party B wishes to lock.
* **`upnlSig`**: A data structure containing the unrealized profit and loss (UPnL) signature necessary for validating the current market conditions.
* **`increaseNonce`**: A boolean indicating whether to increment the nonce for Party B.

**Functionality**:

**Verification**:

* Utilizes `LibMuon.verifyPartyBUpnl` to ensure the UPnL values are accurate, affirming Party B's ability to lock the quote.
* Calls `LibPartyB.checkPartyBValidationToLockQuote` to perform additional checks specific to Party B's requirements for accepting the quote.

**State Updates**:

* Sets the quote's status to `LOCKED`, indicating that Party B has accepted and is now responsible for the trade.
* Updates the `statusModifyTimestamp` to the current block timestamp to log the exact time of the change.
* Assigns Party B as the responsible party by setting `quote.partyB` to the current caller's address.

**Lock Funds**:

* Updates the financial records in `AccountStorage` to reflect that the quote's associated values are now locked under Party B's responsibility. This includes adjusting Party B's pending locked balances to account for the new liabilities assumed with the quote.

**Record Keeping**:

Adds the locked quote to the list of pending quotes for Party B,.

**Storage Interactions**:

Accesses and modifies data within `QuoteStorage` and `AccountStorage` to update the quote's details and financial positions of Party B accordingly.

## unlockQuote()

**Description**: Releases a trading quote from a locked status, potentially reverting it to a pending state if conditions allow, or expiring it if its validity period has passed.

For any given reason, PartyB, having secured the quote, can choose to abandon the opening position. Following the unlocking of the quote, it becomes available for others to secure.

```solidity
    function unlockQuote(uint256 quoteId) internal returns (QuoteStatus) {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();

        Quote storage quote = QuoteStorage.layout().quotes[quoteId];
        require(quote.quoteStatus == QuoteStatus.LOCKED, "PartyBFacet: Invalid state");
        if (block.timestamp > quote.deadline) {
            QuoteStatus result = LibQuote.expireQuote(quoteId);
            return result;
        } else {
            quote.statusModifyTimestamp = block.timestamp;
            quote.quoteStatus = QuoteStatus.PENDING;
            accountLayout.partyBPendingLockedBalances[quote.partyB][quote.partyA].subQuote(quote);
            LibQuote.removeFromPartyBPendingQuotes(quote);
            quote.partyB = address(0);
            return QuoteStatus.PENDING;
        }
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote to be unlocked.

**Functionality**:

**Validation**:

Confirms that the quote is currently in a `LOCKED` state, ensuring that only quotes which are actively committed are considered for unlocking.

**Condition Handling**:

* **Deadline Exceeded**: If the current timestamp exceeds the quote's deadline, the quote is expired. This is handled by `LibQuote.expireQuote`, which adjusts the quote's status.
* **Within Validity**: If the deadline has not passed, the quote is reverted to a `PENDING` state.

**State and Balance Updates**:

* Updates the `statusModifyTimestamp` to the current block timestamp.
* Changes the quote's status back to `PENDING`, indicating it is again open for actions.
* Removes the locked balances associated with the quote from Party B's account, adjusting the financial records to reflect the change in obligations.
* Clears Party B's assignment from the quote, setting `quote.partyB` to the null address to signify no current commitment.

**Return**:

**QuoteStatus**: Returns the new status of the quote, which could either be `PENDING` if the quote has been unlocked and reverted, or the status returned by `expireQuote` if the quote has been expired due to passing its deadline.

**Storage Interactions**:

Modifies data in `QuoteStorage` and `AccountStorage` to update the quote's status, remove financial locks, and clean up any references to Party B's commitment.

## acceptCancelRequest()

**Description**: Processes the acceptance of a cancellation request for a previously locked quote, officially canceling it and adjusting financial records accordingly.

```solidity
    function acceptCancelRequest(uint256 quoteId) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();

        Quote storage quote = QuoteStorage.layout().quotes[quoteId];
        require(quote.quoteStatus == QuoteStatus.CANCEL_PENDING, "PartyBFacet: Invalid state");
        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.CANCELED;
        accountLayout.pendingLockedBalances[quote.partyA].subQuote(quote);
        accountLayout.partyBPendingLockedBalances[quote.partyB][quote.partyA].subQuote(quote);
        // send trading Fee back to partyA
        accountLayout.allocatedBalances[quote.partyA] += LibQuote.getTradingFee(quoteId);

        LibQuote.removeFromPendingQuotes(quote);
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote for which the cancellation request is being accepted.

**Functionality**:

**Validation**:

Ensures that the quote is currently in a `CANCEL_PENDING` state.

**State Updates**:

* Sets the quote's status to `CANCELED`, finalizing the cancellation.
* Updates the `statusModifyTimestamp` to the current block timestamp.

**Financial Adjustments**:

* Removes the locked values associated with the quote from both Party A's and Party B's pending locked balances in `AccountStorage`.
* Credits the trading fee back to Party A's allocated balances, compensating them for the costs associated with the now-canceled trade.

**Record Cleanup**:

Utilizes `LibQuote.removeFromPendingQuotes` to remove the quote from the list of pending quotes.

**Storage Interactions**:

* Modifies `QuoteStorage` to update the quote's status and related details.
* Updates `AccountStorage` to adjust financial balances and remove locked values, reflecting the cancellation of the quote.

## openPosition()

**Description**: Executes the opening of a trading position from a locked or cancel-pending quote. After a quote gets locked by partyB, the position will be opened regarding the given limitations.

#### Partially Opened Positions:

PartyB has the option to open the position with either the full amount requested by the user or a specific fraction of it. For instance, consider a quote for 100 units of a symbol. PartyB could choose to open the position for only 20 units of the total 100, with the remaining units forming a new quote that's available for all PartyBs to act on. The opened position's size can't be excessively small or large. If it's like 99/100, the leftover will be a minuscule quote that falls below the minimum acceptable quote value. Conversely, the position might be so small that it also falls beneath the minimum value. The contract method interface is as follows:

```solidity
    function openPosition(
        uint256 quoteId,
        uint256 filledAmount,
        uint256 openedPrice,
        PairUpnlAndPriceSig memory upnlSig
    ) internal returns (uint256 currentId) {
}
```

**Parameters**:

* **`quoteId`**: Identifier of the quote for which the position is being opened.
* **`filledAmount`**: The amount of the asset involved in the transaction, which may be partial or total of the initial quote quantity.
* **`openedPrice`**: The price at which the position is being opened.
* **`upnlSig`**: Signature data including unrealized profits and losses, ensuring that the transaction meets the required financial conditions.

**Functionality**:

**Validation Checks**:

* Confirms that the quote is either locked or awaiting cancellation but still actionable.
* Ensures that the quote has not expired and that the party initiating the action is not suspended.
* Verifies that the symbol associated with the quote is valid and that the system is not in emergency mode.
* Checks that the filled amount does not exceed the quoted amount and meets minimum quote value requirements after the transaction.

**Financial Adjustments**:

* Applies trading fees to the fee collector's balance based on the filled amount and the quoted price or market price, depending on the order type.
* Adjusts the locked values and balances related to the quote based on whether the entire quote is filled or only partially filled.

**Quote Update and Status Change**:

* Updates the quote's opened price and status to `OPENED`.
* Adjusts the quote's remaining quantity and associated financial obligations if not fully filled.
* Manages the creation of a new quote if the original quote is partially filled.

**Post-Transaction Processing**:

* Registers the quote in the open positions list using `LibQuote.addToOpenPositions`.
* Ensures that the position meets leverage requirements.
* Verifies the solvency post-position opening using `LibSolvency.isSolventAfterOpenPosition`.

**Returns**:

* **`currentId`**: If the quote is partially filled, returns the identifier of the newly created quote; otherwise, returns the identifier of the original quote now in `OPENED` status.

**Storage Interactions**:

* Modifies data in `QuoteStorage`, `AccountStorage`, and other related storages to reflect the new state of the quote and update financial records.

## fillCloseRequest()

**Description**: Completes the closing process for a quote that is either pending close or pending cancellation close. After partyA sends the close request, partyB responds to the request by filling. PartyB can fill the LIMIT requests in multiple steps and each within a different price but the market requests should be filled all at once. The contract method interface is depicted below:

```solidity
    function fillCloseRequest(
        uint256 quoteId,
        uint256 filledAmount,
        uint256 closedPrice,
        PairUpnlAndPriceSig memory upnlSig
    ) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];
        require(
            quote.quoteStatus == QuoteStatus.CLOSE_PENDING ||
                quote.quoteStatus == QuoteStatus.CANCEL_CLOSE_PENDING,
            "PartyBFacet: Invalid state"
        );
        require(block.timestamp <= quote.deadline, "PartyBFacet: Quote is expired");
        if (quote.positionType == PositionType.LONG) {
            require(
                closedPrice >= quote.requestedClosePrice,
                "PartyBFacet: Closed price isn't valid"
            );
        } else {
            require(
                closedPrice <= quote.requestedClosePrice,
                "PartyBFacet: Closed price isn't valid"
            );
        }
        if (quote.orderType == OrderType.LIMIT) {
            require(quote.quantityToClose >= filledAmount, "PartyBFacet: Invalid filledAmount");
        } else {
            require(quote.quantityToClose == filledAmount, "PartyBFacet: Invalid filledAmount");
        }

        LibMuon.verifyPairUpnlAndPrice(upnlSig, quote.partyB, quote.partyA, quote.symbolId);
        LibSolvency.isSolventAfterClosePosition(quoteId, filledAmount, closedPrice, upnlSig);

        accountLayout.partyBNonces[quote.partyB][quote.partyA] += 1;
        accountLayout.partyANonces[quote.partyA] += 1;
        LibQuote.closeQuote(quote, filledAmount, closedPrice);
    }
```

**Parameters**:

* **`quoteId`**: Identifier of the quote being closed.
* **`filledAmount`**: The amount of the asset for which the close request is being fulfilled.
* **`closedPrice`**: The price at which the position is being closed.
* **`upnlSig`**: Signature data containing the unrealized profit and loss, verifying the closing transaction's validity under current market conditions.

**Functionality**:

**Validation Checks**:

* Ensures the quote is in a valid state for closure (`CLOSE_PENDING` or `CANCEL_CLOSE_PENDING`).
* Verifies that the closing operation is being conducted before the expiration of the quote's deadline.
* Confirms that the closed price is appropriate relative to the quote's requested close price, based on the `positionType` (long or short).
* Checks that the filled amount is valid according to the quote's order type (LIMIT or otherwise).

**Market and Financial Validation**:

* Uses `LibMuon.verifyPairUpnlAndPrice` to validate the price and UPnL as per the current market conditions and the involved parties.
* Ensures that closing the position maintains solvency using `LibSolvency.isSolventAfterClosePosition`.

**Transaction Processing**:

* Calls `LibQuote.closeQuote` to officially close the quote with the specified amount and price, updating the system records and financial balances.

**Storage Interactions**:

* Modifies `QuoteStorage` to update the quote's status and details.
* Updates `AccountStorage` to adjust the balances of Party A and Party B involved in the quote.

## acceptCancelCloseRequest()

**Description**: Finalizes the process of canceling a close request on a trading quote, effectively reverting the quote's status from `CANCEL_CLOSE_PENDING` to `OPENED` and resetting associated parameters.

```solidity
    function acceptCancelCloseRequest(uint256 quoteId) internal {
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(
            quote.quoteStatus == QuoteStatus.CANCEL_CLOSE_PENDING,
            "PartyBFacet: Invalid state"
        );
        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.OPENED;
        quote.requestedClosePrice = 0;
        quote.quantityToClose = 0;
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote for which the cancel close request is being accepted.

**Functionality**:

**Validation**:

* Confirms that the quote is in the `CANCEL_CLOSE_PENDING` state.

**Status Update and Reset**:

* Updates the quote’s status to `OPENED`.
* Resets `requestedClosePrice` and `quantityToClose` to zero, clearing the parameters associated with the previously pending close request.

**Storage Interactions**:

Modifies the quote's status and details in `QuoteStorage`, reflecting the cancellation of the close request and the quote's return to open status.

## emergencyClosePosition()

**Description**: This function facilitates the immediate closure of an open trading position or one that is pending closure, under emergency conditions, using the latest market price and UPnL data for validation.

```solidity
    function emergencyClosePosition(uint256 quoteId, PairUpnlAndPriceSig memory upnlSig) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];
        require(quote.quoteStatus == QuoteStatus.OPENED || quote.quoteStatus == QuoteStatus.CLOSE_PENDING, "PartyBFacet: Invalid state");
        LibMuon.verifyPairUpnlAndPrice(upnlSig, quote.partyB, quote.partyA, quote.symbolId);
        uint256 filledAmount = LibQuote.quoteOpenAmount(quote);
        quote.quantityToClose = filledAmount;
        quote.requestedClosePrice = upnlSig.price;
        LibSolvency.isSolventAfterClosePosition(quoteId, filledAmount, upnlSig.price, upnlSig);
        accountLayout.partyBNonces[quote.partyB][quote.partyA] += 1;
        accountLayout.partyANonces[quote.partyA] += 1;
        LibQuote.closeQuote(quote, filledAmount, upnlSig.price);
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote for which the emergency closure is being executed.
* **`upnlSig`**: Signature containing the unrealized profit and loss (UPnL).

**Functionality**:

**Validation**:

* Ensures the quote is either in an `OPENED` or `CLOSE_PENDING` status, suitable for emergency actions.
* Verifies the UPnL and price using `LibMuon.verifyPairUpnlAndPrice`.

**Closure Preparation**:

* Sets the `quantityToClose` to the total open amount of the quote.
* Assigns the `requestedClosePrice` from the UPnL signature, reflecting the current market price at which the closure is to be executed.

**Solvency Checks**:

Performs solvency checks using `LibSolvency.isSolventAfterClosePosition` to ensure closing the position does not render the parties financially unstable or insolvent.
