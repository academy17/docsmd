# View Facet

## Changes:

## Added Utility View Functions

### `getPositionsFilteredByPartyB()`

**Description:** Fetches positions where Party B is involved, filtered by Party B.

```solidity
function getPositionsFilteredByPartyB(
    address partyB,
    uint256 start,
    uint256 size
) external view returns (Quote[] memory)
```

**Parameters:**

* `partyB` - Address of Party B (the solver/hedger).
* `start` - Starting index.
* `size` - Number of positions to retrieve.

**Returns:** An array of `Quote` structs representing positions for Party B.

### `getOpenPositionsFilteredByPartyB()`

**Description:** Fetches open positions where Party B is involved, filtered by Party B.

```solidity
function getOpenPositionsFilteredByPartyB(
    address partyB,
    uint256 start,
    uint256 size
) external view returns (Quote[] memory)
```

**Parameters:**

* `partyB` - Address of Party B (the solver/hedger).
* `start` - Starting index.
* `size` - Number of open positions to retrieve.

**Returns:** An array of `Quote` structs representing open positions for Party B.

### `getActivePositionsFilteredByPartyB()`

**Description:** Fetches active positions where Party B is involved, filtered by Party B.

```solidity
function getActivePositionsFilteredByPartyB(
    address partyB,
    uint256 start,
    uint256 size
) external view returns (Quote[] memory)
```

**Parameters:**

* `partyB` - Address of Party B (the solver/hedger).
* `start` - Starting index.
* `size` - Number of active positions to retrieve.

**Returns:** An array of `Quote` structs representing active positions for Party B.

### `getQuotesWithBitmap()`

**Description:** Fetches quotes using a bitmap filter.

```solidity
function getQuotesWithBitmap(
    Bitmap calldata bitmap,
    uint256 gasNeededForReturn
) external view returns (Quote[] memory quotes)
```

**Parameters:**

* `bitmap` - A `Bitmap` struct used to filter the quotes.
  * `Bitmap` struct:
    * `size` - Size of the bitmap.
    * `elements` - Array of `BitmapElement` structs.
  * `BitmapElement` struct:
    * `offset` - Offset of the bitmap element.
    * `bitmap` - The bitmap value.
* `gasNeededForReturn` - The amount of gas needed for the function to return the quotes.

**Returns:** An array of `Quote` structs representing the filtered quotes.

### Updated ForceClose View Functions

```solidity
// Penalty for partyB that has not functioned properly.
function forceClosePricePenalty() external view returns (uint256)

// In cases where the average price from the timeframe is selected, we also ensure that the length of the timeframe exceeds a certain threshold.
function forceCloseMinSigPeriod() external view returns (uint256)

// Returns forceCloseFirstCooldown, forceCloseSecondCooldown
function forceCloseCooldowns() external view returns (uint256, uint256)

// From
function coolDownsOfMA() external view returns (uint256, uint256, uint256, uint256) {
    return (
        MAStorage.layout().deallocateCooldown,
        MAStorage.layout().forceCancelCooldown,
        MAStorage.layout().forceCancelCloseCooldown,
        MAStorage.layout().forceCloseCooldown
    );
}
// To
function coolDownsOfMA() external view returns (uint256, uint256, uint256, uint256) {
    return (
        MAStorage.layout().deallocateCooldown,
        MAStorage.layout().forceCancelCooldown,
        MAStorage.layout().forceCancelCloseCooldown,
        MAStorage.layout().forceCloseFirstCooldown
    );
}

```

### **`getDeallocateDebounceTime()`**

```solidity
function getDeallocateDebounceTime() external view returns (uint256) {
    return MAStorage.layout().deallocateDebounceTime;
}
```

**Explanation**: Returns the current deallocate debounce time.

### **`forceCloseCooldowns()`**

```solidity
function forceCloseCooldowns() external view returns (uint256, uint256) {
    return (MAStorage.layout().forceCloseFirstCooldown, MAStorage.layout().forceCloseSecondCooldown);
}
```

**Explanation**: Returns the first and second cooldown periods for force closing.

### **`deallocateCooldown()`**

```solidity
function deallocateCooldown() external view returns (uint256) {
    return MAStorage.layout().deallocateCooldown;
}
```

**Explanation**: Returns the current deallocate cooldown period.

### **`getBridgeTransaction()`**

```solidity
function getBridgeTransaction(uint256 transactionId) external view returns (BridgeTransaction memory) {
    return BridgeStorage.layout().bridgeTransactions[transactionId];
}
```

**Explanation**: Returns the details of a bridge transaction specified by `transactionId`.

### **`getNextBridgeTransactionId()`**

```solidity
function getNextBridgeTransactionId() external view returns (uint256) {
    return BridgeStorage.layout().lastId;
}
```

**Explanation**: Returns the ID for the next bridge transaction.

### **`getQuoteCloseId()`**

```solidity
function getQuoteCloseId(uint256 quoteId) external view returns (uint256) {
    return QuoteStorage.layout().closeIds[quoteId];
}
```

**Explanation**: Returns the close ID associated with a specified quote ID.

## **Changed Function Signatures**

### **`pauseState`**

**Before:**

```solidity
function pauseState()
    external
    view
    returns (
        bool globalPaused,
        bool liquidationPaused,
        bool accountingPaused,
        bool partyBActionsPaused,
        bool partyAActionsPaused,
        bool emergencyMode
    )
```

**After:**

```solidity
function pauseState()
    external
    view
    returns (
        bool globalPaused,
        bool liquidationPaused,
        bool accountingPaused,
        bool partyBActionsPaused,
        bool partyAActionsPaused,
        bool internalTransferPaused,
        bool emergencyMode
    )
```

**Explanation**: Added `internalTransferPaused` to the return values to indicate if internal transfers are paused.
