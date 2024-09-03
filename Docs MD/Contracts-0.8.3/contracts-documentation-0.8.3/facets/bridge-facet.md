# Bridge Facet

The Bridge feature allows users to bypass the full cooldown period when withdrawing their money. Certain addresses can be designated as "bridges" within the system. The method `transferToBridge` enables users to quickly transfer the amount they wish to withdraw. The trusted bridge will then send that amount to the user's wallet outside of SYMMIO. After the cooldown period is complete, the bridge can withdraw that amount from SYMMIO. The bridge is responsible for ensuring user compliance with system regulations. If SYMMIO detects any wrongdoing by the user, the bridge transaction will be suspended, and the bridge may become unable to fully withdraw the funds after SYMMIO restores the transaction.

## Added Methods

### `transferToBridge()`

**Description:** Transfers a specified amount to a designated bridge address.

```solidity
function transferToBridge(
    uint256 amount, 
    address bridgeAddress
) external whenNotAccountingPaused notSuspended(msg.sender)
```

**Parameters:**

* `amount` - The amount to transfer to the bridge.
* `bridgeAddress` - The address of the bridge.

**Returns:** None.

**Events Emitted:**

`TransferToBridge(address indexed sender, uint256 amount, address indexed bridgeAddress, uint256 transactionId)`

### `withdrawReceivedBridgeValue()`

**Description:** Withdraws the received bridge value associated with a specific transaction ID.

```solidity
function withdrawReceivedBridgeValue(
    uint256 transactionId
) external whenNotAccountingPaused notSuspended(msg.sender)
```

**Parameters:**

* `transactionId` - The ID of the transaction whose received value is to be withdrawn.

**Returns:** None.

**Events Emitted:**

`WithdrawReceivedBridgeValue(uint256 indexed transactionId)`

### `withdrawReceivedBridgeValues()`

**Description:** Withdraws the received bridge values associated with multiple transaction IDs.

```solidity
function withdrawReceivedBridgeValues(
    uint256[] memory transactionIds
) external whenNotAccountingPaused notSuspended(msg.sender)
```

**Parameters:**

* `transactionIds` - An array of transaction IDs whose received values are to be withdrawn.

**Returns:** None.

**Events Emitted:**

`WithdrawReceivedBridgeValues(uint256[] transactionIds)`

### `suspendBridgeTransaction()`

**Description:** Suspends a specific bridge transaction.

```solidity
function suspendBridgeTransaction(
    uint256 transactionId
) external onlyRole(LibAccessibility.SUSPENDER_ROLE)
```

**Parameters:**

* `transactionId` - The transaction ID of the bridge transaction to be suspended.

**Returns:** None.

**Events Emitted:**

* `SuspendBridgeTransaction(uint256 indexed transactionId)`

### `restoreBridgeTransaction()`

**Description:** Restores a previously suspended bridge transaction and updates the valid transaction amount.

```solidity
function restoreBridgeTransaction(
    uint256 transactionId, 
    uint256 validAmount
) external onlyRole(LibAccessibility.DISPUTE_ROLE)
```

**Parameters:**

* `transactionId` - The transaction ID of the bridge transaction to be restored.
* `validAmount` - The validated amount to be associated with the restored transaction.

**Returns:** None.

**Events Emitted:**

* `RestoreBridgeTransaction(uint256 indexed transactionId, uint256 validAmount)`
