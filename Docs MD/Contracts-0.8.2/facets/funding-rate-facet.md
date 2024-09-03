# Funding Rate Facet

The Funding Rate Facet is responsible for applying funding rates to open positions.

## chargeFundingRate()

The `chargeFundingRate` function is designed to periodically adjust the `openedPrice` of a quote based on funding rates. This function is callled with an array of `quotes` and `rates`. This adjustment reflects the cost or gain from holding positions over time, aligning with market conditions.

```solidity
    function chargeFundingRate(
        address partyA,
        uint256[] memory quoteIds,
        int256[] memory rates,
        PairUpnlSig memory upnlSig
    ) internal {}
```

**Epoch Timestamps and Window Time:**

**`epochDuration`**: This represents the total duration of a funding rate period. It is defined per symbol and dictates how often the funding rate should be applied.

```solidity
epochDuration = SymbolStorage.layout().symbols[quote.symbolId].fundingRateEpochDuration;
```

**`windowTime`**: This is the specific time window within the epoch when the funding rate can be applied. It ensures that funding rate adjustments are made only during this designated period.

```solidity
windowTime = SymbolStorage.layout().symbols[quote.symbolId].fundingRateWindowTime;
```

**Implementation**

```solidity
    function chargeFundingRate(
        address partyA,
        uint256[] memory quoteIds,
        int256[] memory rates,
        PairUpnlSig memory upnlSig
    ) internal {
        LibMuon.verifyPairUpnl(upnlSig, msg.sender, partyA);
        require(quoteIds.length == rates.length && quoteIds.length > 0, "PartyBFacet: Length not match");
        int256 partyBAvailableBalance = LibAccount.partyBAvailableBalanceForLiquidation(
            upnlSig.upnlPartyB,
            msg.sender,
            partyA
        );
        int256 partyAAvailableBalance = LibAccount.partyAAvailableBalanceForLiquidation(
            upnlSig.upnlPartyA,
            partyA
        );
        uint256 epochDuration;
        uint256 windowTime;
        for (uint256 i = 0; i < quoteIds.length; i++) {
            Quote storage quote = QuoteStorage.layout().quotes[quoteIds[i]];
            require(quote.partyA == partyA, "PartyBFacet: Invalid quote");
            require(quote.partyB == msg.sender, "PartyBFacet: Sender isn't partyB of quote");
            require(
                quote.quoteStatus == QuoteStatus.OPENED ||
                    quote.quoteStatus == QuoteStatus.CLOSE_PENDING ||
                    quote.quoteStatus == QuoteStatus.CANCEL_CLOSE_PENDING,
                "PartyBFacet: Invalid state"
            );
            epochDuration = SymbolStorage.layout().symbols[quote.symbolId].fundingRateEpochDuration;
            require(epochDuration > 0, "PartyBFacet: Zero funding epoch duration");
            windowTime = SymbolStorage.layout().symbols[quote.symbolId].fundingRateWindowTime;
            uint256 latestEpochTimestamp = (block.timestamp / epochDuration) * epochDuration;
            uint256 paidTimestamp;
            if (block.timestamp <= latestEpochTimestamp + windowTime) {
                require(
                    latestEpochTimestamp > quote.lastFundingPaymentTimestamp,
                    "PartyBFacet: Funding already paid for this window"
                );
                paidTimestamp = latestEpochTimestamp;
            } else {
                uint256 nextEpochTimestamp = latestEpochTimestamp + epochDuration;
                require(
                    block.timestamp >= nextEpochTimestamp - windowTime,
                    "PartyBFacet: Current timestamp is out of window"
                );
                require(
                    nextEpochTimestamp > quote.lastFundingPaymentTimestamp,
                    "PartyBFacet: Funding already paid for this window"
                );
                paidTimestamp = nextEpochTimestamp;
            }
            if (rates[i] >= 0) {
                require(
                    uint256(rates[i]) <= quote.maxFundingRate,
                    "PartyBFacet: High funding rate"
                );
                uint256 priceDiff = (quote.openedPrice * uint256(rates[i])) / 1e18;
                if (quote.positionType == PositionType.LONG) {
                    quote.openedPrice += priceDiff;
                } else {
                    quote.openedPrice -= priceDiff;
                }
                partyAAvailableBalance -= int256(LibQuote.quoteOpenAmount(quote) * priceDiff / 1e18);
                partyBAvailableBalance += int256(LibQuote.quoteOpenAmount(quote) * priceDiff / 1e18);
            } else {
                require(
                    uint256(-rates[i]) <= quote.maxFundingRate,
                    "PartyBFacet: High funding rate"
                );
                uint256 priceDiff = (quote.openedPrice * uint256(-rates[i])) / 1e18;
                if (quote.positionType == PositionType.LONG) {
                    quote.openedPrice -= priceDiff;
                } else {
                    quote.openedPrice += priceDiff;
                }
                partyAAvailableBalance += int256(LibQuote.quoteOpenAmount(quote) * priceDiff / 1e18);
                partyBAvailableBalance -= int256(LibQuote.quoteOpenAmount(quote) * priceDiff / 1e18);
            }
            quote.lastFundingPaymentTimestamp = paidTimestamp;
        }
        require(partyAAvailableBalance >= 0, "PartyBFacet: PartyA will be insolvent");
        require(partyBAvailableBalance >= 0, "PartyBFacet: PartyB will be insolvent");
        AccountStorage.layout().partyBNonces[msg.sender][partyA] += 1;
        AccountStorage.layout().partyANonces[partyA] += 1;
    }
}
```

For each quote, the function retrieves the current settings from `SymbolStorage`, including the `epochDuration` and `windowTime`. The `latestEpochTimestamp` is calculated by dividing the current block timestamp by the `epochDuration` and multiplying back by `epochDuration`. This calculation finds the start of the current epoch. The function then checks if the current block time is within the window to apply funding rates based on the `latestEpochTimestamp` and `windowTime`.

**Adjusting the Open Price**:

Depending on whether the position is long or short, the open price is adjusted upwards or downwards by a proportion of the open price multiplied by the funding rate (`rate[i]`). The adjustment is either added to or subtracted from the `openedPrice` based on the position type (long or short). Balances for Party A and Party B are updated to reflect the monetary impact of the price adjustment.

**Post-Adjustment Checks**:

After all quotes have been processed and adjusted, the function checks to ensure that neither Party A nor Party B becomes insolvent as a result of the funding rate application.

**Update Nonces and Balances**:

Nonces in `AccountStorage` are incremented to prevent replay attacks and ensure state consistency. The balances are updated to reflect the applied funding rates.
