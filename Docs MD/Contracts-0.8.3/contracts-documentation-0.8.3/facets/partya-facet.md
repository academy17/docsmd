# PartyA Facet

## Changes:

### Introduced an Enhanced ForceClose Process

The previous implementation of force close was limited, allowing users to force close their positions only if the price at that moment was higher or lower than the requested price, depending on the type of position. The new version introduces a new signature from Muon called `HighLowPriceSig`, which specifies a timeframe with `startTime` and `endTime`. This period must align with specified cooldown periods to ensure that the data represents a stable and acceptable range for partyB to act upon. Additionally, for this timeframe, the minimum, maximum, and average prices of the symbol are required.

The calculation of the close price within the `forceClosePosition` function depends on the type of position (LONG or SHORT) and the prices provided in the `HighLowPriceSig`. For a LONG position, the function checks if the highest price achieved is sufficient to cover the requested close price plus a gap ratio. If this condition is met, the close price is set as the requested close price plus a penalty ratio, ensuring that partyB incurs a cost for not closing the position themselves on time. This close price is then compared against the average price, and the higher of these two prices is chosen as the final price for executing the close.

Conversely, for a SHORT position, the function evaluates whether the lowest price is less than or equal to the requested close price minus the gap ratio. If this condition is met, the close price is determined as the requested close price minus the penalty ratio. This price is then compared against the average price, and the lower of these two prices is selected as the close price.

**Changed Methods:**

```solidity
// From
function forceClosePosition(uint256 quoteId, PairUpnlAndPriceSig memory upnlSig)
// To
function forceClosePosition(uint256 quoteId, HighLowPriceSig memory sig)

```

### **`sendQuote()`**&#x20;

**Before:**

```solidity
function sendQuote(
    address[] memory partyBsWhiteList,
    uint256 symbolId,
    PositionType positionType,
    OrderType orderType,
    uint256 price,
    uint256 quantity,
    uint256 cva,
    uint256 lf,
    uint256 partyAmm,
    uint256 partyBmm,
    uint256 maxFundingRate,
    uint256 deadline,
    SingleUpnlAndPriceSig memory upnlSig
) internal returns (uint256 currentId) {
    ...
    accountLayout.allocatedBalances[msg.sender] -= LibQuote.getTradingFee(currentId);
}
```

**After:**

```solidity
function sendQuote(
    address[] memory partyBsWhiteList,
    uint256 symbolId,
    PositionType positionType,
    OrderType orderType,
    uint256 price,
    uint256 quantity,
    uint256 cva,
    uint256 lf,
    uint256 partyAmm,
    uint256 partyBmm,
    uint256 maxFundingRate,
    uint256 deadline,
    address affiliate,
    SingleUpnlAndPriceSig memory upnlSig
) internal returns (uint256 currentId) {
    ...
    require(maLayout.affiliateStatus[affiliate], "PartyAFacet: Invalid affiliate");
    ...
    uint256 fee = LibQuote.getTradingFee(currentId);
    accountLayout.allocatedBalances[msg.sender] -= fee;
    emit SharedEvents.BalanceChangePartyA(msg.sender, fee, SharedEvents.BalanceChangeType.PLATFORM_FEE_OUT);
}
```

**Explanation**:

* Added `affiliate` parameter to the `sendQuote` function.
* Added a validation step to check the status of the `affiliate`.
* Emitted `BalanceChangePartyA` event after deducting the trading fee.

### **`requestToClosePosition()`**

**Before:**

```solidity
function requestToClosePosition(
    uint256 quoteId,
    uint256 closePrice,
    uint256 quantityToClose,
    OrderType orderType,
    uint256 deadline
) internal {
    ...
}
```

**After:**

```solidity
function requestToClosePosition(
    uint256 quoteId,
    uint256 closePrice,
    uint256 quantityToClose,
    OrderType orderType,
    uint256 deadline
) internal {
    SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
    QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
    Quote storage quote = quoteLayout.quotes[quoteId];
    ...
    quoteLayout.closeIds[quoteId] = ++quoteLayout.lastCloseId;
}
```

**Explanation**: To simplify tracking for partyBs, every close request now includes an ID. This ID is included in all types of close events. Note that this ID is not stored historically; each new close request will overwrite the previous ID.

#### Events Updated with `closeId`

The `closeId` is now added to the following events:

* `FillCloseRequest`
* `ExpireQuote`
* `AcceptCancelCloseRequest`
* `EmergencyClosePosition`
* `RequestToClosePosition`
* `RequestToCancelCloseRequest`
* `ForceCancelCloseRequest`
* `ForceClosePosition`
* `LiquidatePositionsPartyA`
* `LiquidatePositionsPartyB`

