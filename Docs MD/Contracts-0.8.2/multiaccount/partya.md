# PartyA

The `SymmioPartyA` contract serves as a proxy for users to interact with the Symmio platform via their respective sub-accounts. Each sub-account, represented by an instance of `SymmioPartyA`, can forward calls to the Symmio diamond. Here's a detailed breakdown of the functionalities and features provided by this contract:

### **Constructor**

Initializes a new `SymmioPartyA` contract with specific roles and addresses.

```solidity
    constructor(address admin, address multiAccountAddress, address symmioAddress_) {
        _grantRole(DEFAULT_ADMIN_ROLE, admin);
        _grantRole(MULTIACCOUNT_ROLE, multiAccountAddress);
        symmioAddress = symmioAddress_;
    }
```

* **Parameters**:
  * `admin`: The administrator of the contract who is granted the `DEFAULT_ADMIN_ROLE`.
  * `multiAccountAddress`: The address of the `MultiAccount` contract, granted the `MULTIACCOUNT_ROLE` to allow it to make calls on behalf of the sub-account.
  * `symmioAddress_`: The initial address of the Symmio platform contract to which all calls will be forwarded.
* **Operations**:
  * Grants `DEFAULT_ADMIN_ROLE` to `admin`.
  * Grants `MULTIACCOUNT_ROLE` to `multiAccountAddress`.
  * Sets the initial Symmio platform address (this is the Diamond).

### **setSymmioAddress()**

Updates the address of the Symmio platform contract. Only an account with `DEFAULT_ADMIN_ROLE` can invoke this function.

```solidity

    function setSymmioAddress(address symmioAddress_) external onlyRole(DEFAULT_ADMIN_ROLE) {
        emit SetSymmioAddress(symmioAddress, symmioAddress_);
        symmioAddress = symmioAddress_;
    }
```

**Parameters**:

* `symmioAddress_`: The new address of the Symmio platform.

**Operations**:

* Emits a `SetSymmioAddress` event containing the old and new addresses.
* Updates the `symmioAddress` state variable.

**Events**:

```solidity
    event SetSymmioAddress(address oldV3ContractAddress, address newV3ContractAddress);
```

### **\_call()**

Forwards calls from the `SymmioPartyA` contract to the Symmio platform. This method is crucial for interacting with the Symmio functionalities through sub-accounts. Only an account with `MULTIACCOUNT_ROLE` can invoke this function, meaning only the `MultiAccount` contract can direct actions.

```solidity
    function _call(bytes memory _callData) external onlyRole(MULTIACCOUNT_ROLE) returns (bool _success, bytes memory _resultData) {
        return symmioAddress.call{ value: 0 }(_callData);
    }
```

**Parameters**:

* `_callData`: The bytecode of the function call to be forwarded to the Symmio platform.

**Returns**:

* `_success`: A boolean indicating whether the call was successful.
* `_resultData`: The data returned from the call if successful, or error information if the call failed.
