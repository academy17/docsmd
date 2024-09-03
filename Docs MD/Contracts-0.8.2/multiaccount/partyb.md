# PartyB

The `SymmioPartyB` contract acts as a management contract for solvers or "Party B" . This contract allows for controlled interaction with the Symmio platform and external addresses for managing roles, permissions, and .

### **Constructor**

Sets up initial state, disabling initializers to ensure proper contract upgrades.

```solidity
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }
```

* **Operations**: Calls `_disableInitializers()` to prepare the contract for the upgrade-safe initialization pattern.

### **initialize()**

Properly initializes the contract with roles and the address of the Symmio platform.

```solidity
    function initialize(address admin, address symmioAddress_) public initializer {
        __Pausable_init();
        __AccessControl_init();

        _grantRole(DEFAULT_ADMIN_ROLE, admin);
        _grantRole(TRUSTED_ROLE, admin);
        _grantRole(MANAGER_ROLE, admin);
        symmioAddress = symmioAddress_;
    }
```

**Parameters**:

* `admin`: The administrator's address that will receive initial roles.
* `symmioAddress_`: The address of the Symmio platform this contract will interact with.

**Operations**:

* Initializes the contract as pausable and with access control features.
* Grants `DEFAULT_ADMIN_ROLE`, `TRUSTED_ROLE`, `MANAGER_ROLE`, `PAUSER_ROLE`, and `UNPAUSER_ROLE` to the admin.
* Sets the initial Symmio address.

### **setSymmioAddress()**

Updates the address of the Symmio platform to which calls are forwarded.

```solidity
    function setSymmioAddress(address addr) external onlyRole(DEFAULT_ADMIN_ROLE) {
        emit SetSymmioAddress(symmioAddress, addr);
        symmioAddress = addr;
    }
```

**Access Control**: Only accessible by users with `DEFAULT_ADMIN_ROLE`.

**Parameters**:

* `addr`: New address for the Symmio platform.

**Operations**: Updates `symmioAddress` and emits the `SetSymmioAddress` event.

### **setRestrictedSelector()**

Manages access control by setting restrictions on specific function selectors.

```solidity
    function setRestrictedSelector(bytes4 selector, bool state) external onlyRole(DEFAULT_ADMIN_ROLE) {
        restrictedSelectors[selector] = state;
        emit SetRestrictedSelector(selector, state);
    }

```

**Access Control**: Requires `DEFAULT_ADMIN_ROLE`.

**Parameters**:

* `selector`: The function selector to restrict or unrestrict.
* `state`: Boolean value indicating whether the selector is restricted.

**Operations**: Updates the `restrictedSelectors` mapping and emits the `SetRestrictedSelector` event.

### **setMulticastWhitelist()**

Manages a whitelist of addresses allowed to be called through multicasting. This allows functions on multiple contracts to be called at the same time.

```solidity
    function setMulticastWhitelist(address addr, bool state) external onlyRole(MANAGER_ROLE) {
        require(addr != address(this), "SymmioPartyB: Invalid address");
        multicastWhitelist[addr] = state;
        emit SetMulticastWhitelist(addr, state);
    }
```

* **Access Control**: Requires `MANAGER_ROLE`.
* **Parameters**:
  * `addr`: The address to whitelist or remove from whitelist.
  * `state`: Boolean indicating whether to add (true) or remove (false) from the whitelist.
* **Operations**: Updates the `multicastWhitelist` mapping and emits the `SetMulticastWhitelist` event.

### **\_approve()**

Approves a token amount to the Symmio address. Only accessible by users with `TRUSTED_ROLE` and when the contract is not paused.

```solidity
    function _approve(address token, uint256 amount) external onlyRole(TRUSTED_ROLE) whenNotPaused {
        require(IERC20Upgradeable(token).approve(symmioAddress, amount), "SymmioPartyB: Not approved");
    }
```

**Parameters**:

* `token`: ERC20 token address.
* `amount`: Amount of tokens to approve.

### **\_executeCall()**

Executes a call to a specified address with provided call data.

```solidity
    function _executeCall(address destAddress, bytes memory callData) internal {
        require(destAddress != address(0), "SymmioPartyB: Invalid address");
        require(callData.length >= 4, "SymmioPartyB: Invalid call data");

        if (destAddress == symmioAddress) {
            bytes4 functionSelector;
            assembly {
                functionSelector := mload(add(callData, 0x20))
            }
            if (restrictedSelectors[functionSelector]) {
                _checkRole(MANAGER_ROLE, msg.sender);
            } else {
                require(hasRole(MANAGER_ROLE, msg.sender) || hasRole(TRUSTED_ROLE, msg.sender), "SymmioPartyB: Invalid access");
            }
        } else {
            require(multicastWhitelist[destAddress], "SymmioPartyB: Destination address is not whitelisted");
            _checkRole(TRUSTED_ROLE, msg.sender);
        }

        (bool success, ) = destAddress.call{ value: 0 }(callData);
        require(success, "SymmioPartyB: Execution reverted");
    }

```

* **Parameters**:
  * `destAddress`: Destination address for the call.
  * `callData`: The data to be sent in the call.

This function ensures that the destination address is not zero and that the call data is at least 4 bytes (minimum size for a function selector).

Depending on the function selector extracted from the call data:

* If the function is restricted (as per `restrictedSelectors`), the caller must have a `MANAGER_ROLE`.
* Otherwise, either the `MANAGER_ROLE` or `TRUSTED_ROLE` is required.

&#x20;If the destination address is not the Symmio platform, it checks whether the address is whitelisted for calls. The call is made using the low-level `call` function, and successful execution is required; otherwise, it reverts.

### **\_call()**

Executes multiple calls to the Symmio address. Iterates over `_callDatas` and executes each using `_executeCall`.

```solidity
    function _call(bytes[] calldata _callDatas) external whenNotPaused {
        for (uint8 i; i < _callDatas.length; i++) _executeCall(symmioAddress, _callDatas[i]);
    }
```

**Parameters**:

* `_callDatas`: An array of call data bytes to be executed.

### **\_multicastCall()**

Enables the solver to interact with multiple contracts in a single transaction.

```solidity
    function _multicastCall(address[] calldata destAddresses, bytes[] calldata _callDatas) external whenNotPaused {
        require(destAddresses.length == _callDatas.length, "SymmioPartyB: Array length mismatch");

        for (uint8 i; i < _callDatas.length; i++) _executeCall(destAddresses[i], _callDatas[i]);
    }
```

It checks that the arrays of destination addresses and call data have the same length to ensure each address has corresponding call data then iterates through each address and associated call data, executing each via the `_executeCall` function.

### **withdrawERC20()**

Allows withdrawal of ERC20 tokens to the caller's address. Requires `MANAGER_ROLE` and transfers specified amount of tokens to the caller.

```solidity
    function withdrawERC20(address token, uint256 amount) external onlyRole(MANAGER_ROLE) {
        require(IERC20Upgradeable(token).transfer(msg.sender, amount), "SymmioPartyB: Not transferred");
    }
```

* **Parameters**:
  * `token`: The ERC20 token address.
  * `amount`: The amount of tokens to withdraw.

### **pause() and unpause()**

Controls the pausing and unpausing of the contract. Requires `PAUSER_ROLE` for pausing and `UNPAUSER_ROLE` for unpausing.

```solidity
    function pause() external onlyRole(PAUSER_ROLE) {
        _pause();
    }

    function unpause() external onlyRole(UNPAUSER_ROLE) {
        _unpause();
    }
```
