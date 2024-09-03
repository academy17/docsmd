# Account Facet

## Changes:

### Added `InternalTransfer` Method

This method allows a user to send their deposits to another user’s allocated balance. The receiver address cannot be a partyB and must not be in a liquidated state. This enables users to transfer funds between accounts without waiting for `deallocateCooldown`.

```solidity
function internalTransfer(address user, uint256 amount) internal {
    AccountStorage.Layout storage accountLayout = AccountStorage.layout();
    require(
        accountLayout.allocatedBalances[user] + amount <= GlobalAppStorage.layout().balanceLimitPerUser,
        "AccountFacet: Allocated balance limit reached"
    );
    require(accountLayout.balances[msg.sender] >= amount, "AccountFacet: Insufficient balance");
    accountLayout.balances[msg.sender] -= amount;
    accountLayout.allocatedBalances[user] += amount;
}
```

Symmio can pause internal transfers when needed.

### **`deallocate()`**

**Before:**

```solidity
require(
    accountLayout.allocatedBalances[msg.sender] >= amount,
    "AccountFacet: Insufficient allocated Balance"
);
```

**After:**

```solidity
require(
    block.timestamp >= accountLayout.withdrawCooldown[msg.sender] + MAStorage.layout().deallocateDebounceTime,
    "AccountFacet: Too many deallocate in a short window"
);
require(
    accountLayout.allocatedBalances[msg.sender] >= amount,
    "AccountFacet: Insufficient allocated Balance"
);
```

**Explanation**: Added a new condition to enforce a debounce time between deallocation requests to prevent too frequent deallocations.

#### Summary of Changes

* **Added a debounce time check** to the `deallocate` function to ensure that deallocation requests are not made too frequently. This is done by checking that the current block timestamp is greater than or equal to the sum of the user's `withdrawCooldown` and the global `deallocateDebounceTime`.
* **Maintained the existing check** to ensure the user has sufficient allocated balance for the deallocation request.

This change helps to prevent abuse of the deallocation process by enforcing a time window between consecutive deallocations.
