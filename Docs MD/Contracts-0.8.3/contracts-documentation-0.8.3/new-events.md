# New Events

## New Events

## `BalanceChange`

To facilitate the tracking of user balances for partyB or anyone monitoring the Symmio contracts, we have added an event called `BalanceChanged` that is emitted every time a user's allocated balance changes. The reason for the change can be accessed through the type field of this event.

```solidity
enum BalanceChangeType {
ALLOCATE,
DEALLOCATE,
PLATFORM_FEE_IN,
PLATFORM_FEE_OUT,
REALIZED_PNL_IN,
REALIZED_PNL_OUT,
CVA_IN,
CVA_OUT,
LF_IN,
LF_OUT
}

event BalanceChangePartyA(address indexed partyA, uint256 amount, BalanceChangeType _type);

event BalanceChangePartyB(address indexed partyB, address indexed partyA, uint256 amount, BalanceChangeType _type);
```

## **ExpireQuote**

To enhance clarity, the single `ExpireQuote` event has been split into two separate events: `ExpireQuoteOpen` and `ExpireQuoteClose`.

### **`ExpireQuoteOpen`**

```solidity
event ExpireQuoteOpen(QuoteStatus quoteStatus, uint256 quoteId);
```

**Explanation**: Emitted when an open request expires, providing the quote status and quote ID.

### **`ExpireQuoteClose`**

```solidity
event ExpireQuoteClose(QuoteStatus quoteStatus, uint256 quoteId, uint256 closeId);
```

**Explanation**: Emitted when a close request expires, including the quote status, quote ID, and close ID.

## **Affiliate Events**

### **`RegisterAffiliate`**

```solidity
event RegisterAffiliate(address affiliate);
```

**Explanation**: Emitted when an affiliate is registered.

### **`DeregisterAffiliate`**

```solidity
event DeregisterAffiliate(address affiliate);
```

**Explanation**: Emitted when an affiliate is deregistered.

## **Bridge Events**

### **`AddBridge`**

```solidity
event AddBridge(address bridge);
```

**Explanation**: Emitted when a bridge is added.

### **`RemoveBridge`**

```solidity
eevent RemoveBridge(address bridge);
```

**Explanation**: Emitted when a bridge is removed.

### **`TransferToBridge`**

```solidity
event TransferToBridge(address user, uint256 amount, address bridgeAddress, uint256 transactionId);
```

**Explanation**: Emitted when a transfer to a bridge is initiated.

### **`WithdrawReceivedBridgeValue`**

```solidity
sevent WithdrawReceivedBridgeValue(uint256 transactionId);
```

**Explanation**: Emitted when the received bridge value is withdrawn.

### **`SuspendBridgeTransaction`**

```solidity
event SuspendBridgeTransaction(uint256 transactionId);
```

**Explanation**: Emitted when a bridge transaction is suspended.

### **`RestoreBridgeTransaction`**

```solidity
event RestoreBridgeTransaction(uint256 transactionId, uint256 validAmount);
```

**Explanation**: Emitted when a bridge transaction is restored.

### **`WithdrawReceivedBridgeValues`**

```solidity
event WithdrawReceivedBridgeValues(uint256[] transactionIds);
```

**Explanation**: Emitted when multiple received bridge values are withdrawn.

### **`SetInvalidBridgedAmountsPool`**

```solidity
event SetInvalidBridgedAmountsPool(address oldInvalidBridgedAmountsPool, address newInvalidBridgedAmountsPool);
```

**Explanation**: Emitted when the invalid bridged amounts pool is updated.

## **Internal Transfer Events**

### **`InternalTransfer`**

```solidity
event InternalTransfer(address sender, address user, uint256 userNewAllocatedBalance, uint256 amount);
```

**Explanation**: Emitted when an internal transfer is made.

## **Deferred Liquidation Events**

### **`DeferredLiquidatePartyA`**

```solidity
event DeferredLiquidatePartyA(
    address liquidator,
    address partyA,
    uint256 allocatedBalance,
    int256 upnl,
    int256 totalUnrealizedLoss,
    bytes liquidationId,
    uint256 liquidationBlockNumber,
    uint256 liquidationTimestamp,
    uint256 liquidationAllocatedBalance
);
```

**Explanation**: Emitted when a deferred liquidation for Party A occurs.
