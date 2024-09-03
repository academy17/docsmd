# PartyA Facet

The PartyA Facet contains functions related to partyA actions. These include opening and closing quotes, as well as modifying positions.

## sendQuote()

The user’s request to open a position is called a quote.

This function creates and registers a new quote for trading, based on the parameters specified by Party A. It involves locking values, validating conditions, and ensuring compliance with system requirements.

```solidity
function sendQuote(
   address[] memory partyBsWhiteList,
   uint256 symbolId,
   PositionType positionType,
   OrderType orderType,
   uint256 price,
   uint256 quantity,
   uint256 cva,
   uint256 mm,
   uint256 lf,
   uint256 maxFundingRate,
   uint256 deadline,
   SingleUpnlAndPriceSig memory upnlSig
);
```

**Parameters**:

* **`partyBsWhiteList`**: An array of whitelisted partyB's permitted to accept the quote.
* **`symbolId`**: Each symbol within the system possesses a unique identifier, for instance, BTCUSDT carries its own distinct ID
* **`positionType`**: Can be **SHORT** or **LONG** (0 or 1)
* **`orderType`**: Can be **LIMIT** or **MARKET** (0 or 1)
* **`price`**: For limit orders, this is the user-requested price for the position, and for market orders, this acts as the price threshold that the user is willing to open a position. For example, if the market price for an arbitrary symbol is $1000 and the user wants to open a short position on this symbol they might be ok with prices up to $990
* **`quantity`**: Size of the position
* **`cva`**: Credit Valuation Adjustment. Either partyA or partyB can get liquidated and CVA is the penalty that the liquidated side should pay to the other one
* **`mm`**: Maintenance Margin. The amount that is actually behind the position and is considered in liquidation status
* **`lf`**: Liquidation Fee. It is the prize that will be paid to the liquidator user
* **`maxFundingRate`**: Max funding rate
* **`deadline`**: The user should set a deadline for their request. If no PartyB takes action on the quote within this timeframe, the request will expire (further details about the expiration procedure will be provided later)
*   **`upnlSig`**: The Muon signature for user upnl and symbol price

    \*Every symbol has a minimum acceptable quote value that should be acknowledged when issuing a quote. For instance, one cannot open a position on BTCUSDT that is less than a certain number of dollars.

**Functionality**:

**Balance Checks**: Ensures that Party A has sufficient available balance to cover the quote and associated trading fees.

**Quote Creation**:

* Constructs a `Quote` object with initial values and locked parameters.
* Registers the quote in `QuoteStorage`, updates pending quotes and locked balances.

**Event Triggering**: Emits relevant events to log the creation of the quote and the locking of funds.

**Storage Interactions**:

Updates multiple storage layouts (`QuoteStorage`, `AccountStorage`, `SymbolStorage`) to record the new quote and adjust financial balances and settings accordingly.

**Return**:

**`currentId`**: The identifier of the newly created quote, which can be used for reference in subsequent transactions or queries.

## requestToCancelQuote()

**Description**: Handles requests to cancel a trading quote based on its current status and applicable rules. The function adjusts the quote's status and manages associated balances based on whether the quote can be immediately canceled or needs to enter a cancellation pending state.

```solidity
    function requestToCancelQuote(uint256 quoteId) internal returns (QuoteStatus result) {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(
            quote.quoteStatus == QuoteStatus.PENDING || quote.quoteStatus == QuoteStatus.LOCKED,
            "PartyAFacet: Invalid state"
        );

        if (block.timestamp > quote.deadline) {
            result = LibQuote.expireQuote(quoteId);
        } else if (quote.quoteStatus == QuoteStatus.PENDING) {
            quote.quoteStatus = QuoteStatus.CANCELED;
            accountLayout.allocatedBalances[quote.partyA] += LibQuote.getTradingFee(quote.id);
            accountLayout.pendingLockedBalances[quote.partyA].subQuote(quote);
            LibQuote.removeFromPartyAPendingQuotes(quote);
            result = QuoteStatus.CANCELED;
        } else {
            // Quote is locked
            quote.quoteStatus = QuoteStatus.CANCEL_PENDING;
            result = QuoteStatus.CANCEL_PENDING;
        }
        quote.statusModifyTimestamp = block.timestamp;
    }
```

**Parameters**:

* **`quoteId`**: The unique identifier of the quote to be cancelled.

**Functionality**:

**Validation**: Ensures that the quote is in an appropriate state for cancellation (either `PENDING` or `LOCKED`).

**Cancellation Logic**:

* **Expired Quotes**: If the current time exceeds the quote's deadline, it triggers an expiration process which typically adjusts the quote's status to expired and manages any associated balances or fees.
* **Pending Quotes**: If the quote is still pending and not past its deadline, it's directly cancelled. This involves reverting any locked values and fees associated with the quote.
* **Locked Quotes**: If the quote is locked, it transitions to a `CANCEL_PENDING` state.

**State and Balance Updates**:

* Updates the quote's status to either `CANCELED` or `CANCEL_PENDING`.
* Adjusts Party A's allocated and pending locked balances according to the new quote status.
* Updates the modification timestamp to reflect the time of the status change.

**Return**:

* **`result`**: The new status of the quote post-request, indicating whether it has been cancelled, set to cancel pending, or processed in some other way.

**Storage Interactions**:

Accesses and modifies data in `QuoteStorage` and `AccountStorage` to update the quote's status and manage financial balances accordingly.

## requestToClosePosition()

**Description**: This function processes requests to close or partially close an existing open trading position based on specified parameters. It adjusts the quote's status to reflect a pending close operation and updates relevant details such as the close price and quantity.

```solidity
    function requestToClosePosition(
        uint256 quoteId,
        uint256 closePrice,
        uint256 quantityToClose,
        OrderType orderType,
        uint256 deadline
    ) internal {
        SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(quote.quoteStatus == QuoteStatus.OPENED, "PartyAFacet: Invalid state");
        require(deadline >= block.timestamp, "PartyAFacet: Low deadline");
        require(
            LibQuote.quoteOpenAmount(quote) >= quantityToClose,
            "PartyAFacet: Invalid quantityToClose"
        );

        // check that remaining position is not too small
        if (LibQuote.quoteOpenAmount(quote) > quantityToClose) {
            require(
                ((LibQuote.quoteOpenAmount(quote) - quantityToClose) * quote.lockedValues.totalForPartyA()) /
                    LibQuote.quoteOpenAmount(quote) >=
                    symbolLayout.symbols[quote.symbolId].minAcceptableQuoteValue,
                "PartyAFacet: Remaining quote value is low"
            );
        }

        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.CLOSE_PENDING;
        quote.requestedClosePrice = closePrice;
        quote.quantityToClose = quantityToClose;
        quote.orderType = orderType;
        quote.deadline = deadline;
    }
```

**Parameters**:

* **`quoteId`**: The identifier of the quote associated with the open position.
* **`closePrice`**: In the case of limit orders, this is the price the user wants to close the position at. For market orders, it's more like a price threshold the user's okay with when closing their position. Say, for a random symbol, the market price is $1000. If a user wants to close a short position on this symbol, they might be cool with prices up to $1010
* **`quantityToClose`**: The amount of the position that the requester wishes to close.
* **`orderType`**: Specifies the type of order (**MARKET** or **LIMIT**) that is being requested for the close.
* **`deadline`**: The timestamp by which the close operation must be executed.

**Validation**:

This function ensures that the quote is currently in an `OPENED` state, eligible for closing. Checks that the current time is before the specified deadline. It verifies that the quantity to be closed does not exceed the currently open amount.

**Position Size Check**:

If not closing the entire position, ensures that the remaining open amount still meets the minimum acceptable quote value criteria.

**State Updates**:

* Sets the quote's status to `CLOSE_PENDING`, indicating that a closure request has been initiated.
* Records the requested close price, the quantity to close, the type of order, and updates the deadline for the close operation.

**Timestamp Management**:

Updates the `statusModifyTimestamp` to the current block timestamp to log the time of the modification.

**Storage Interactions**:

Accesses and modifies data in `QuoteStorage` and `SymbolStorage` to manage the quote details and check system-wide financial constraints.

## requestToCancelCloseRequest()

**Description**: If the user has already sent the close request but partyB has not filled it yet, the user can request to cancel it. PartyB can either accept the cancel request or fill the close request ignoring the user's request. The contract method interface for the user is as below:

```solidity
    function requestToCancelCloseRequest(uint256 quoteId) internal returns (QuoteStatus) {
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(quote.quoteStatus == QuoteStatus.CLOSE_PENDING, "PartyAFacet: Invalid state");
        if (block.timestamp > quote.deadline) {
            LibQuote.expireQuote(quoteId);
            return QuoteStatus.OPENED;
        } else {
            quote.statusModifyTimestamp = block.timestamp;
            quote.quoteStatus = QuoteStatus.CANCEL_CLOSE_PENDING;
            return QuoteStatus.CANCEL_CLOSE_PENDING;
        }
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote for which the close request cancellation is being processed.

**Functionality**:

**Validation**:

Ensures that the quote is currently in a `CLOSE_PENDING` state, indicating that there is an ongoing request to close the position.

**Condition Handling**:

* **Deadline Passed**: If the current time exceeds the deadline set for the close request, the quote is expired using the `LibQuote.expireQuote` method, and its status is reverted to `OPENED`.
* **Deadline Active**: If the deadline has not yet passed, the quote's status is updated to `CANCEL_CLOSE_PENDING`.

**Timestamp Update**:

The `statusModifyTimestamp` is updated to the current block timestamp to log the exact time of the status change.

**Returns**:

**`QuoteStatus`**: Returns the new status of the quote, which can be either `OPENED` if the quote was expired, or `CANCEL_CLOSE_PENDING` if the close request cancellation is now pending.

**Storage Interactions**:

Modifies data in `QuoteStorage` related to the specific quote, updating its status and the timestamp of the last modification.

## forceCancelQuote()

**Description**: This function forcefully cancels a quote that is in a `CANCEL_PENDING` state if the required cooldown period has elapsed. It adjusts account balances and removes locked values associated with the quote.

```solidity
    function forceCancelQuote(uint256 quoteId) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(quote.quoteStatus == QuoteStatus.CANCEL_PENDING, "PartyAFacet: Invalid state");
        require(
            block.timestamp > quote.statusModifyTimestamp + maLayout.forceCancelCooldown,
            "PartyAFacet: Cooldown not reached"
        );
        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.CANCELED;
        accountLayout.pendingLockedBalances[quote.partyA].subQuote(quote);
        accountLayout.partyBPendingLockedBalances[quote.partyB][quote.partyA].subQuote(quote);

        // send trading Fee back to partyA
        accountLayout.allocatedBalances[quote.partyA] += LibQuote.getTradingFee(quote.id);

        LibQuote.removeFromPendingQuotes(quote);
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote to be forcefully cancelled.

**Functionality**:

**Validation**:

* Checks that the quote is currently in the `CANCEL_PENDING` state, ensuring it's eligible for cancellation.
* Verifies that the requisite cooldown period (`forceCancelCooldown` from `MAStorage`) has passed since the last status modification, allowing for the force cancellation.

**State and Balance Updates**:

* Updates the quote's status to `CANCELED`, marking it as no longer active.
* Removes the quote's locked values from both the `partyA`'s and `partyB`'s pending locked balances in `AccountStorage`.
* Returns the trading fee previously allocated for the quote back to `partyA`'s allocated balance.

**Housekeeping**:

Calls `LibQuote.removeFromPendingQuotes` to clean up any references to the quote from pending lists, ensuring data consistency and freeing up resources.

**Timestamp Update**:

The `statusModifyTimestamp` is updated to the current block timestamp to record the exact time of the cancellation.

**Storage Interactions**:

Modifies multiple data structures within `AccountStorage` and `QuoteStorage` to update financial balances and quote statuses accordingly.

## forceCancelQuote()

**Description**: This function forcefully cancels a quote that is in a `CANCEL_PENDING` state if the required cooldown period has elapsed. It adjusts account balances and removes locked values associated with the quote.

```solidity
    function forceCancelQuote(uint256 quoteId) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        require(quote.quoteStatus == QuoteStatus.CANCEL_PENDING, "PartyAFacet: Invalid state");
        require(
            block.timestamp > quote.statusModifyTimestamp + maLayout.forceCancelCooldown,
            "PartyAFacet: Cooldown not reached"
        );
        quote.statusModifyTimestamp = block.timestamp;
        quote.quoteStatus = QuoteStatus.CANCELED;
        accountLayout.pendingLockedBalances[quote.partyA].subQuote(quote);
        accountLayout.partyBPendingLockedBalances[quote.partyB][quote.partyA].subQuote(quote);

        // send trading Fee back to partyA
        accountLayout.allocatedBalances[quote.partyA] += LibQuote.getTradingFee(quote.id);

        LibQuote.removeFromPendingQuotes(quote);
    }

```

**Parameters**:

* **quoteId**: The identifier of the quote to be forcefully cancelled.

**Functionality**:

**Validation**:

* Checks that the quote is currently in the `CANCEL_PENDING` .
* Verifies that the requisite cooldown period (`forceCancelCooldown` from `MAStorage`) has passed since the last status modification, allowing for the force cancellation.

**State and Balance Updates**:

* Updates the quote's status to `CANCELED`, marking it as no longer active.
* Removes the quote's locked values from both the `partyA`'s and `partyB`'s pending locked balances in `AccountStorage`.
* Returns the trading fee previously allocated for the quote back to `partyA`'s allocated balance.

**Housekeeping**:

Calls `LibQuote.removeFromPendingQuotes` to clean up any references to the quote from pending lists.

**Timestamp Update**:

The `statusModifyTimestamp` is updated to the current block timestamp to record the exact time of the cancellation.

**Storage Interactions**:

Modifies multiple data structures within `AccountStorage` and `QuoteStorage` to update balances and quote statuses accordingly.

## forceClosePosition()

The force close mechanism is designed to enable users to close their positions if certain conditions are met. This mechanism comes into play when a hedger does not fill a close order at the specified price.&#x20;

```solidity
    function forceClosePosition(uint256 quoteId, PairUpnlAndPriceSig memory upnlSig) internal {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        Quote storage quote = QuoteStorage.layout().quotes[quoteId];

        uint256 filledAmount = quote.quantityToClose;
        require(quote.quoteStatus == QuoteStatus.CLOSE_PENDING, "PartyAFacet: Invalid state");
        require(
            block.timestamp > quote.statusModifyTimestamp + maLayout.forceCloseCooldown,
            "PartyAFacet: Cooldown not reached"
        );
        require(block.timestamp <= quote.deadline, "PartyBFacet: Quote is expired");
        require(
            quote.orderType == OrderType.LIMIT,
            "PartyBFacet: Quote's order type should be LIMIT"
        );
        if (quote.positionType == PositionType.LONG) {
            require(
                upnlSig.price >=
                    quote.requestedClosePrice +
                        (quote.requestedClosePrice * maLayout.forceCloseGapRatio) /
                        1e18,
                "PartyAFacet: Requested close price not reached"
            );
        } else {
            require(
                upnlSig.price <=
                    quote.requestedClosePrice -
                        (quote.requestedClosePrice * maLayout.forceCloseGapRatio) /
                        1e18,
                "PartyAFacet: Requested close price not reached"
            );
        }

        LibMuon.verifyPairUpnlAndPrice(upnlSig, quote.partyB, quote.partyA, quote.symbolId);
        LibSolvency.isSolventAfterClosePosition(
            quoteId,
            filledAmount,
            upnlSig.price,
            upnlSig
        );
        accountLayout.partyANonces[quote.partyA] += 1;
        accountLayout.partyBNonces[quote.partyB][quote.partyA] += 1;
        LibQuote.closeQuote(quote, filledAmount, upnlSig.price);
    }

```

**Parameters**:

* **`quoteId`**: The identifier of the quote associated with the position to be closed.
* **`upnlSig`**: A data structure containing signatures for unrealized profit and loss (UPnL) and the price at which the closure is executed, used for verification.

**Functionality**:

**Validation**:

* Ensures that the quote is in a `CLOSE_PENDING` state.
* Checks that the required cooldown period (`forceCloseCooldown` from `MAStorage`) has passed since the last modification to allow for a force close.
* Verifies that the current time is within the deadline set for the position closure.
* Confirms that the quote’s order type is `LIMIT`, which is necessary for processing a forced close.
* Depending on whether the position is long or short, checks if the market price (`upnlSig.price`) meets the conditions adjusted by the `forceCloseGapRatio` for a forceful closure.

**Market Price Evaluation**:

* For a long position, the function ensures the current price is at least the requested close price plus a calculated gap (`forceCloseGapRatio`). For a short position, it verifies the price is at or below the requested price minus the gap.

**Execution**:

* Verifies the authenticity of the UPnL and price data using `LibMuon.verifyPairUpnlAndPrice`.
* Calls `LibSolvency.isSolventAfterClosePosition` to ensure closing the position does not lead to insolvency.
* Updates nonce counters for both parties involved.
* Uses `LibQuote.closeQuote` helper function to close the quote with the specified amount and price.

**Storage Interactions**:

Updates various data fields within `QuoteStorage` and `AccountStorage`, reflecting the new state of the quote and the updated financial positions of the involved parties.
