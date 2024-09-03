# Liquidation Facet (Implementation)

## Changes:

### **`liquidatePendingPositionsPartyA()`**

**Updated Return Type**:

Updated to return `uint256[] memory liquidatedAmounts, bytes memory liquidationId`.

**Updated Code**:

```solidity
accountLayout.partyAReimbursement[partyA] += LibQuote.getTradingFee(quote.id);
quote.quoteStatus = QuoteStatus.LIQUIDATED_PENDING;
```

**Updated Functionality**:

* Added a new pending state `QuoteStatus.LIQUIDATED_PENDING`.

### **`liquidatePositionsPartyA()`**

**Updated Return Type**:

Updated to return `bool, uint256[] memory liquidatedAmounts, uint256[] memory closeIds, bytes memory liquidationId`.

**Updated Code**:

```solidity
liquidatedAmounts[index] = quote.quantity - quote.closedAmount;
closeIds[index] = quoteLayout.closeIds[quote.id];
quote.quoteStatus = QuoteStatus.LIQUIDATED;
```

**Updated Functionality**:

* Added `closeIds` and `liquidatedAmounts` to capture detailed information about liquidated quotes.
* Changed state to `QuoteStatus.LIQUIDATED`.

### **`resolveLiquidationDispute()`**

**Updated Return Type**:

Updated to return `bytes memory`.

**Updated Code**:

```solidity
accountLayout.settlementStates[partyA][partyBs[i]].actualAmount = amounts[i];
return accountLayout.liquidationDetails[partyA].liquidationId;
```

### **`settlePartyALiquidation()`**

**Updated Return Type**:

Updated to return `int256[] memory settleAmounts, bytes memory liquidationId`.

**Updated Code**:

```solidity
emit SharedEvents.BalanceChangePartyA(partyA, accountLayout.settlementStates[partyA][partyB].cva, SharedEvents.BalanceChangeType.CVA_OUT);
emit SharedEvents.BalanceChangePartyB(partyB, partyA, accountLayout.settlementStates[partyA][partyB].cva, SharedEvents.BalanceChangeType.CVA_IN);
```

####
