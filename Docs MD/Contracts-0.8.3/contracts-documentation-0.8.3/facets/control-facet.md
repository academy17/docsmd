# Control Facet

## Changes:

### Make `forceCloseGapRatio` per symbol

Different symbols with different volatility require different gap ratios for forceClose.

**Updated Methods:**

```solidity
// From
function setForceCloseGapRatio(uint256 forceCloseGapRatio) external onlyRole(LibAccessibility.SETTER_ROLE)
// To
function setForceCloseGapRatio(uint256 symbolId, uint256 forceCloseGapRatio) external onlyRole(LibAccessibility.SETTER_ROLE)

// From
function forceCloseGapRatio() external view returns (uint256)
// To
function forceCloseGapRatio(uint256 symbolId) external view returns (uint256)
```

### &#x20;Added Affiliates into the System

All affiliates, such as intentX, Based, Befi, Core, etc., are now registered in the core contracts. Each affiliate can have a separate `feeCollector`. The new method `sendQuoteWithAffiliate` is introduced, and the old `sendQuote` method is deprecated and will likely be removed in the next version.

* The `affiliate` field is added to the quote struct.
* `AFFILIATE_MANAGER_ROLE` is introduced, granting control over adding new affiliates and changing their feeCollectors.

**Added Methods:**

```solidity
function registerAffiliate(address affiliate) external onlyRole(LibAccessibility.AFFILIATE_MANAGER_ROLE)

function deregisterAffiliate(address affiliate) external onlyRole(LibAccessibility.AFFILIATE_MANAGER_ROLE)

function isAffiliate(address affiliate) external view returns (bool)
```

**Updated Methods:**

```solidity
// From
function setFeeCollector(address feeCollector) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE)
// To
function setFeeCollector(address affiliate, address feeCollector) external onlyRole(LibAccessibility.AFFILIATE_MANAGER_ROLE)

// From
function getFeeCollector() external view returns (address)
// To
function getFeeCollector(address affiliate) external view returns (address)
```

The old `feeCollector` storage is now renamed to `defaultFeeCollectors` and will be used as a fallback when a collector hasn’t been set for an affiliate. Corresponding setters and getters are also added for that.

### **Added `setDeallocateDebounceTime()`**

```solidity
function setDeallocateDebounceTime(uint256 deallocateDebounceTime) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetDeallocateDebounceTime(MAStorage.layout().deallocateDebounceTime, deallocateDebounceTime);
    MAStorage.layout().deallocateDebounceTime = deallocateDebounceTime;
}
```

**Explanation**: This function allows setting the deallocation debounce time. It emits an event `SetDeallocateDebounceTime` indicating the old and new debounce times.
