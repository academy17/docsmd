# PartyB Facet

## Changes:

### **`acceptCancelRequest()`**

**Before:**

```solidity
// send trading Fee back to partyA
accountLayout.allocatedBalances[quote.partyA] += LibQuote.getTradingFee(quoteId);
```

**After:**

```solidity
// send trading Fee back to partyA
uint256 fee = LibQuote.getTradingFee(quoteId);
accountLayout.allocatedBalances[quote.partyA] += fee;
emit SharedEvents.BalanceChangePartyA(quote.partyA, fee, SharedEvents.BalanceChangeType.PLATFORM_FEE_IN);
```

**Explanation**: Added emission of `BalanceChangePartyA` event when returning the trading fee to Party A upon quote cancellation.

### **`openPosition()`**

**Before:**

```solidity
accountLayout.balances[GlobalAppStorage.layout().feeCollector] +=
    (filledAmount * quote.requestedOpenPrice * quote.tradingFee) / 1e36;
...
accountLayout.balances[GlobalAppStorage.layout().feeCollector] +=
    (filledAmount * quote.marketPrice * quote.tradingFee) / 1e36;
...
accountLayout.pendingLockedBalances[quote.partyA].subQuote(quote);
...
accountLayout.allocatedBalances[newQuote.partyA] += LibQuote.getTradingFee(newQuote.id);
...
accountLayout.pendingLockedBalances[quote.partyA].sub(filledLockedValues);
...
newQuote.lockedValues = quote.lockedValues.sub(filledLockedValues);
```

**After:**

```solidity
accountLayout.balances[GlobalAppStorage.layout().affiliateFeeCollector[quote.affiliate]] +=
    (filledAmount * quote.requestedOpenPrice * quote.tradingFee) / 1e36;
...
accountLayout.balances[GlobalAppStorage.layout().affiliateFeeCollector[quote.affiliate]] +=
    (filledAmount * quote.marketPrice * quote.tradingFee) / 1e36;
...
uint256 fee = LibQuote.getTradingFee(newQuote.id);
accountLayout.allocatedBalances[newQuote.partyA] += fee;
emit SharedEvents.BalanceChangePartyA(newQuote.partyA, fee, SharedEvents.BalanceChangeType.PLATFORM_FEE_IN);
...
accountLayout.pendingLockedBalances[quote.partyA].sub(filledLockedValues);
emit SharedEvents.BalanceChangePartyA(quote.partyA, filledLockedValues.totalForPartyA(), SharedEvents.BalanceChangeType.LOCK_OUT);
...
newQuote.lockedValues = quote.lockedValues.sub(filledLockedValues);
emit SharedEvents.BalanceChangePartyA(newQuote.partyA, filledLockedValues.totalForPartyA(), SharedEvents.BalanceChangeType.LOCK_OUT);
```

**Explanation**: Added emission of `BalanceChangePartyA` in all places where the allocated balance of the user changes. Also, platform fees are now paid to fee collectors of each affiliate separately.

### **`emergencyClosePosition()`**

**Before:**

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

**After:**

```solidity
function emergencyClosePosition(uint256 quoteId, PairUpnlAndPriceSig memory upnlSig) internal {
    AccountStorage.Layout storage accountLayout = AccountStorage.layout();
    Quote storage quote = QuoteStorage.layout().quotes[quoteId];
    Symbol memory symbol = SymbolStorage.layout().symbols[quote.symbolId];
    require(
        GlobalAppStorage.layout().emergencyMode || GlobalAppStorage.layout().partyBEmergencyStatus[quote.partyB] || !symbol.isValid,
        "PartyBFacet: Operation not allowed. Either emergency mode must be active, party B must be in emergency status, or the symbol must be delisted"
    );
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

**Explanation**: From time to time, Binance delists some of its pairs. In the current version, we had to put partyBs in emergency mode to let them close their positions without user requests. In the next version, if a symbol becomes invalid, partyB can emergency close all its positions.
