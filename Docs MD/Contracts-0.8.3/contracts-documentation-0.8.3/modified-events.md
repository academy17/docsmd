# Modified Events

## **Accounting Events**

### **`AllocatePartyA`**

**Old Event**:

```solidity
event AllocatePartyA(address user, uint256 amount);
```

**New Event**:

```solidity
event AllocatePartyA(address user, uint256 amount, uint256 newAllocatedBalance);
```

### **`DeallocatePartyA`**&#x20;

**Old Event**:

```solidity
event DeallocatePartyA(address user, uint256 amount);
```

**New Event**:

```solidity
event DeallocatePartyA(address user, uint256 amount, uint256 newAllocatedBalance);
```

### **`AllocateForPartyB`**&#x20;

**Old Event**:

```solidity
event AllocateForPartyB(address partyB, address partyA, uint256 amount);
```

**New Event**:

```solidity
event AllocateForPartyB(address partyB, address partyA, uint256 amount, uint256 newAllocatedBalance);
```

### **`DeallocateForPartyB`**&#x20;

**Old Event**:

```solidity
event DeallocateForPartyB(address partyB, address partyA, uint256 amount);
```

**New Event**:

```solidity
event DeallocateForPartyB(address partyB, address partyA, uint256 amount, uint256 newAllocatedBalance);
```

### **`TransferAllocation`**&#x20;

**Old Event**:

```solidity
event TransferAllocation(uint256 amount, address origin, address recipient);
```

**New Event**:

```solidity
event TransferAllocation(uint256 amount, address origin, uint256 originNewAllocatedBalance, address recipient, uint256 recipientNewAllocatedBalance);
```

## **Control Events**

### **`SetMuonConfig`**&#x20;

**Old Event**:

```solidity
event SetMuonConfig(uint256 upnlValidTime, uint256 priceValidTime, uint256 priceQuantityValidTime);
```

**New Event**:

```solidity
event SetMuonConfig(uint256 upnlValidTime, uint256 priceValidTime);
```

### **`SetFeeCollector`**&#x20;

**Old Event**:

```solidity
event SetFeeCollector(address oldFeeCollector, address newFeeCollector);
```

**New Event**:

```solidity
event SetFeeCollector(address affiliate, address oldFeeCollector, address newFeeCollector);
```

### **`SetForceCloseCooldown`**&#x20;

**Old Event**:

```solidity
event SetForceCloseCooldown(uint256 oldForceCloseCooldown, uint256 newForceCloseCooldown);
```

**New Event**:

```solidity
event SetForceCloseCooldowns(uint256 oldForceCloseFirstCooldown, uint256 newForceCloseFirstCooldown, uint256 oldForceCloseSecondCooldown, uint256 newForceCloseSecondCooldown);
```

### **`SetForceCloseGapRatio`**&#x20;

**Old Event**:

```solidity
event SetForceCloseGapRatio(uint256 oldForceCloseGapRatio, uint256 newForceCloseGapRatio);
```

**New Event**:

```solidity
event SetForceCloseGapRatio(uint256 symbolId, uint256 oldForceCloseGapRatio, uint256 newForceCloseGapRatio);
```

## **Liquidation Events**

### **`SetSymbolsPrices`**&#x20;

**Old Event**:

```solidity
event SetSymbolsPrices(address liquidator, address partyA, uint256[] symbolIds, uint256[] prices);
```

**New Event**:

```solidity
event SetSymbolsPrices(address liquidator, address partyA, uint256[] symbolIds, uint256[] prices, bytes liquidationId);
```

### **`LiquidatePartyA`**&#x20;

**Old Event**:

```solidity
event LiquidatePartyA(address liquidator, address partyA, uint256 allocatedBalance, int256 upnl, int256 totalUnrealizedLoss);
```

**New Event**:

```solidity
event LiquidatePartyA(address liquidator, address partyA, uint256 allocatedBalance, int256 upnl, int256 totalUnrealizedLoss, bytes liquidationId);
```

### **`LiquidatePositionsPartyA`**&#x20;

**Old Event**:

```solidity
event LiquidatePositionsPartyA(address liquidator, address partyA, uint256[] quoteIds);
```

**New Event**:

```solidity
event LiquidatePositionsPartyA(address liquidator, address partyA, uint256[] quoteIds, uint256[] liquidatedAmounts, uint256[] closeIds, bytes liquidationId);
```

### **`LiquidatePendingPositionsPartyA`**&#x20;

**Old Event**:

```solidity
event LiquidatePendingPositionsPartyA(address liquidator, address partyA);
```

**New Event**:

```solidity
event LiquidatePendingPositionsPartyA(address liquidator, address partyA, uint256[] quoteIds, uint256[] liquidatedAmounts, bytes liquidationId);
```

### **`SettlePartyALiquidation`**&#x20;

**Old Event**:

```solidity
event SettlePartyALiquidation(address partyA, address[] partyBs, int256[] amounts);
```

**New Event**:

```solidity
event SettlePartyALiquidation(address partyA, address[] partyBs, int256[] amounts, bytes liquidationId);
```

### **`LiquidationDisputed`**&#x20;

**Old Event**:

```solidity
event LiquidationDisputed(address partyA);
```

**New Event**:

```solidity
event LiquidationDisputed(address partyA, bytes liquidationId);
```

### **`ResolveLiquidationDispute`**&#x20;

**Old Event**:

```solidity
event ResolveLiquidationDispute(address partyA, address[] partyBs, int256[] amounts, bool disputed);
```

**New Event**:

```solidity
event ResolveLiquidationDispute(address partyA, address[] partyBs, int256[] amounts, bool disputed, bytes liquidationId);
```

### **`FullyLiquidatedPartyA`**&#x20;

**Old Event**:

```solidity
event FullyLiquidatedPartyA(address partyA);
```

**New Event**:

```solidity
event FullyLiquidatedPartyA(address partyA, bytes liquidationId);
```

### **`LiquidatePositionsPartyB`**&#x20;

**Old Event**:

```solidity
event LiquidatePositionsPartyB(address liquidator, address partyB, address partyA, uint256[] quoteIds);
```

**New Event**:

```solidity
event LiquidatePositionsPartyB(address liquidator, address partyB, address partyA, uint256[] quoteIds, uint256[] liquidatedAmounts, uint256[] closeIds);
```

## **Close Events**

### **`RequestToClosePosition`**&#x20;

**Old Event**:

```solidity
event RequestToClosePosition(address partyA, address partyB, uint256 quoteId, uint256 closePrice, uint256 quantityToClose, OrderType orderType, uint256 deadline, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event RequestToClosePosition(address partyA, address partyB, uint256 quoteId, uint256 closePrice, uint256 quantityToClose, OrderType orderType, uint256 deadline, QuoteStatus quoteStatus, uint256 closeId);
```

### **`RequestToCancelCloseRequest`**&#x20;

**Old Event**:

```solidity
event RequestToCancelCloseRequest(address partyA, address partyB, uint256 quoteId, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event RequestToCancelCloseRequest(address partyA, address partyB, uint256 quoteId, QuoteStatus quoteStatus, uint256 closeId);
```

### **`ForceCancelCloseRequest`**&#x20;

**Old Event**:

```solidity
event ForceCancelCloseRequest(uint256 quoteId, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event ForceCancelCloseRequest(uint256 quoteId, QuoteStatus quoteStatus, uint256 closeId);
```

### **`ForceClosePosition`**&#x20;

**Old Event**:

```solidity
event ForceClosePosition(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event ForceClosePosition(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus, uint256 closeId);
```

### **`FillCloseRequest`**&#x20;

**Old Event**:

```solidity
event FillCloseRequest(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event FillCloseRequest(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus, uint256 closeId);
```

### **`AcceptCancelCloseRequest`**&#x20;

**Old Event**:

```solidity
event AcceptCancelCloseRequest(uint256 quoteId, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event AcceptCancelCloseRequest(uint256 quoteId, QuoteStatus quoteStatus, uint256 closeId);
```

### **`EmergencyClosePosition`**&#x20;

**Old Event**:

```solidity
event EmergencyClosePosition(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus);
```

**New Event**:

```solidity
event EmergencyClosePosition(uint256 quoteId, address partyA, address partyB, uint256 filledAmount, uint256 closedPrice, QuoteStatus quoteStatus, uint256 closeId);
```
