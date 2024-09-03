# Control Facet

The `ControlFacet` serves as a critical component for administrative control and operational management within the system. This facet allows authorized users to perform essential functions such as transferring ownership, managing roles, and registering or deregistering entities. It also provides mechanisms to adjust operational parameters like cooldowns, penalties, and configuration settings for associated contracts and storage structures. By centralizing these capabilities, the `ControlFacet` ensures that administrative tasks are performed securely and efficiently.

## **Function Overview**

### **transferOwnership()**

**Description**: Transfers the ownership of the Diamond contract to another address. `LibDiamond.setContractOwner` emits an `OwnershipTransferred` event.

```solidity
    function transferOwnership(address owner) external onlyOwner {
        require(owner != address(0), "ControlFacet: Zero address");
        LibDiamond.setContractOwner(owner);
    }
```

**Input Parameters**:&#x20;

* `owner`: the address of the new owner.

**Storage Interactions**: Updates the `ds.contractOwner`  in `DiamondStorage`.

**Event Structure**

```solidity
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
```

### **setAdmin()**

**Description**: Grants an address with the `DEFAULT_ADMIN_ROLE`.

```solidity
    function setAdmin(address user) external onlyOwner {
        require(user != address(0), "ControlFacet: Zero address");
        GlobalAppStorage.layout().hasRole[user][LibAccessibility.DEFAULT_ADMIN_ROLE] = true;
        emit RoleGranted(LibAccessibility.DEFAULT_ADMIN_ROLE, user);
    }
```

**Input Parameters**:&#x20;

* `user`: the address of the new admin.

**Storage Interactions**: Updates the `hasRole` mapping in `GlobalAppStorage`.

**Event Structure**

```solidity
    event RoleGranted(bytes32 role, address user);
```

### **grantRole()**

**Description**: Grants an address with a bytes32`role`.

```solidity
    function grantRole(
        address user,
        bytes32 role
    ) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        require(user != address(0), "ControlFacet: Zero address");
        GlobalAppStorage.layout().hasRole[user][role] = true;
        emit RoleGranted(role, user);
    }
```

**Input Parameters**:&#x20;

* `user`: the address of the new admin.
* `role`: the keccak256 hash of the role.

**Storage Interactions**: Updates the `hasRole` mapping in `GlobalAppStorage`.

**Event Structure**

```solidity
    event RoleGranted(bytes32 role, address user);
```

### revokeRole`()`

**Description**: Revokes a specified role from an address. This function ensures that the specified user no longer holds the privileges associated with the role.

```solidity
function revokeRole(
    address user,
    bytes32 role
) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
    GlobalAppStorage.layout().hasRole[user][role] = false;
    emit RoleRevoked(role, user);
}
```

**Input Parameters**:

* **user**: The address from which the role is to be revoked.
* **role**: The `bytes32` hash of the role being revoked.

**Storage Interactions**:

* Updates the `hasRole` mapping in `GlobalAppStorage`, setting the specified role for the given user to `false`.

**Event Structure**:

```solidity
    event RoleRevoked(bytes32 role, address user);
```

### registerPartyB()

**Description**: Registers an address as "Party B" within the system.&#x20;

```solidity
function registerPartyB(
    address partyB
) external onlyRole(LibAccessibility.PARTY_B_MANAGER_ROLE) {
    require(partyB != address(0), "ControlFacet: Zero address");
    require(
        !MAStorage.layout().partyBStatus[partyB],
        "ControlFacet: Address is already registered"
    );
    MAStorage.layout().partyBStatus[partyB] = true;
    MAStorage.layout().partyBList.push(partyB);
    emit RegisterPartyB(partyB);
}
```

**Input Parameters**:

* **`partyB`**: The address to be registered as Party B.

**Storage Interactions**:

* Updates the `partyBStatus` mapping in `MAStorage` to `true`, indicating that the address is registered as Party B.
* Adds the address to the `partyBList` in `MAStorage`.

**Event Structure**:

```solidity
    event RegisterPartyB(address partyB);
```

### deregisterPartyB()

**Description**: Removes an address from the registered "Party B" list within the system.&#x20;

```solidity
function deregisterPartyB(
    address partyB,
    uint256 index
) external onlyRole(LibAccessibility.PARTY_B_MANAGER_ROLE) {
    require(partyB != address(0), "ControlFacet: Zero address");
    require(MAStorage.layout().partyBStatus[partyB], "ControlFacet: Address is not registered");
    require(MAStorage.layout().partyBList[index] == partyB, "ControlFacet: Invalid index");
    uint256 lastIndex = MAStorage.layout().partyBList.length - 1;
    require(index <= lastIndex, "ControlFacet: Invalid index");
    MAStorage.layout().partyBStatus[partyB] = false;
    MAStorage.layout().partyBList[index] = MAStorage.layout().partyBList[lastIndex];
    MAStorage.layout().partyBList.pop();
    emit DeregisterPartyB(partyB, index);
}
```

**Input Parameters**:

* **`partyB`**: The address to be deregistered as Party B.
* **`index`**: The position of the address in the `partyBList` to ensure accurate and safe removal.

**Storage Interactions**:

* Sets `partyBStatus` for the address in `MAStorage` to `false`, indicating that it is no longer registered.
* Replaces the entry at the specified `index` in `partyBList` with the last item in the list and then removes the last item.

**Event Structure**:

```solidity
    event DeregisterPartyB(address partyB, uint256 index);
```

### setMuonConfig()

**Description**: Configures the operational parameters related to the Muon network.

```solidity
function setMuonConfig(
    uint256 upnlValidTime,
    uint256 priceValidTime,
    uint256 priceQuantityValidTime
) external onlyRole(LibAccessibility.MUON_SETTER_ROLE) {
    emit SetMuonConfig(upnlValidTime, priceValidTime, priceQuantityValidTime);
    MuonStorage.Layout storage muonLayout = MuonStorage.layout();
    muonLayout.upnlValidTime = upnlValidTime;
    muonLayout.priceValidTime = priceValidTime;
    muonLayout.priceQuantityValidTime = priceQuantityValidTime;
}
```

**Input Parameters**:

* **`upnlValidTime`**: The duration that the UPnL data remains valid.
* **`priceValidTime`**: The duration that the price data is considered valid.
* **`priceQuantityValidTime`**: The duration for which the price quantity data remains valid.

**Storage Interactions**:

* Updates the `upnlValidTime`, `priceValidTime`, and `priceQuantityValidTime` in `MuonStorage`.&#x20;

**Event Structure**:

```solidity
event SetMuonConfig(uint256 upnlValidTime, uint256 priceValidTime, uint256 priceQuantityValidTime);
```

### setMuonIds()

**Description**: Sets the identifiers and gateway address associated with the Muon network configuration for the system. These settings are vital for securing and validating interactions with the Muon network.

```solidity
function setMuonIds(
    uint256 muonAppId,
    address validGateway,
    PublicKey memory publicKey
) external onlyRole(LibAccessibility.MUON_SETTER_ROLE) {
    MuonStorage.Layout storage muonLayout = MuonStorage.layout();
    muonLayout.muonAppId = muonAppId;
    muonLayout.validGateway = validGateway;
    muonLayout.muonPublicKey = publicKey;
    emit SetMuonIds(muonAppId, validGateway, publicKey.x, publicKey.parity);
}
```

**Input Parameters**:

* **muonAppId**: The application ID for the Muon network.
* **validGateway**: The address of the gateway used for Muon network validations.
* **publicKey**: A `PublicKey` struct containing the elliptic curve public key (`x` coordinate and `parity`) used for cryptographic operations in the Muon network.

**Storage Interactions**:

* Updates the `muonAppId`, `validGateway`, and `muonPublicKey` in `MuonStorage`. These parameters are crucial for ensuring that the system communicates correctly and securely with the Muon network.

**Event Structure**:

```solidity
event SetMuonIds(uint256 muonAppId, address validGateway, uint256 publicKeyX, bool publicKeyParity);
```

### setCollateral()

**Description**: Configures the primary collateral type used within the system. This function sets a specific ERC20 token as the collateral for transactions and operations.&#x20;

```solidity
function setCollateral(
    address collateral
) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
    require(collateral != address(0), "ControlFacet: Zero address");
    require(
        IERC20Metadata(collateral).decimals() <= 18,
        "ControlFacet: Token with more than 18 decimals not allowed"
    );
    if (GlobalAppStorage.layout().collateral != address(0)) {
        require(
            IERC20Metadata(GlobalAppStorage.layout().collateral).balanceOf(address(this)) == 0,
            "ControlFacet: There is still collateral in the contract"
        );
    }
    GlobalAppStorage.layout().collateral = collateral;
    emit SetCollateral(collateral);
}
```

**Input Parameters**:

* **`collateral`**: The address of the ERC20 token to be set as collateral.

**Storage Interactions**:

* Validates the current balance of any previously set collateral to ensure it is zero before proceeding.
* Updates the `collateral` address in `GlobalAppStorage` to the new token address specified, ensuring that all future transactions use the newly specified token as collateral.

**Event Structure**:

```solidity
event SetCollateral(address collateral);
```

### addSymbol()

**Description**: Registers a new trading symbol within the system. This function sets variables such as the minimum quote value, leverage, trading fee, and the parameters governing the funding rate.&#x20;

```solidity
function addSymbol(
    string memory name,
    uint256 minAcceptableQuoteValue,
    uint256 minAcceptablePortionLF,
    uint256 tradingFee,
    uint256 maxLeverage,
    uint256 fundingRateEpochDuration,
    uint256 fundingRateWindowTime
) public onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    require(
        fundingRateWindowTime < fundingRateEpochDuration / 2,
        "ControlFacet: High window time"
    );
    require(tradingFee <= 1e18, "ControlFacet: High trading fee");
    uint256 lastId = ++SymbolStorage.layout().lastId;
    Symbol memory symbol = Symbol(
        lastId,
        name,
        true,
        minAcceptableQuoteValue,
        minAcceptablePortionLF,
        tradingFee,
        maxLeverage,
        fundingRateEpochDuration,
        fundingRateWindowTime
    );
    SymbolStorage.layout().symbols[lastId] = symbol;
    emit AddSymbol(
        lastId,
        name,
        minAcceptableQuoteValue,
        minAcceptablePortionLF,
        tradingFee, 
        maxLeverage,
        fundingRateEpochDuration,
        fundingRateWindowTime
    );
}
```

**Input Parameters**:

* **`name`**: The name of the symbol to be registered.
* **`minAcceptableQuoteValue`**: The minimum value of a quote for the symbol to be considered valid.
* **`minAcceptablePortionLF`**: The minimum portion assigned as a liquidator fee, awarded to the liquidator.
* **`tradingFee`**: The fee charged per trade.
* **`maxLeverage`**: The maximum leverage allowed for trading this symbol.
* **`fundingRateEpochDuration`**: The duration of each funding rate epoch, during which interest rates are recalculated.
* **`fundingRateWindowTime`**: The window of time within an epoch when the funding rate can be changed.

**Storage Interactions**:

* Increments the `lastId` in `SymbolStorage` to assign a unique ID to the new symbol.
* Stores the new symbol parameters in the `symbols` mapping of `SymbolStorage`, associating them with the newly incremented symbol ID.

**Event Structure**:

```solidity
event AddSymbol(
    uint256 indexed symbolId,
    string name,
    uint256 minAcceptableQuoteValue,
    uint256 minAcceptablePortionLF,
    uint256 tradingFee,
    uint256 maxLeverage,
    uint256 fundingRateEpochDuration,
    uint256 fundingRateWindowTime
);
```

### addSymbols()

**Description**: Facilitates the batch registration of multiple trading symbols within the system. This function leverages the `addSymbol` function to register each symbol in the provided array.

```solidity
function addSymbols(
    Symbol[] memory symbols
) external onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    for (uint8 i; i < symbols.length; i++) {
        addSymbol(
            symbols[i].name,
            symbols[i].minAcceptableQuoteValue,
            symbols[i].minAcceptablePortionLF,
            symbols[i].tradingFee,
            symbols[i].maxLeverage,
            symbols[i].fundingRateEpochDuration,
            symbols[i].fundingRateWindowTime
        );
    }
}
```

**Input Parameters**:

* **symbols**: An array of `Symbol` structs, each containing the necessary parameters to register a new trading symbol. Each `Symbol` struct includes:\

  * **`name`**: The name of the symbol to be registered.
  * **`minAcceptableQuoteValue`**: The minimum value of a quote for the symbol to be considered valid.
  * **`minAcceptablePortionLF`**: The minimum portion assigned as a liquidator fee, awarded to the liquidator.
  * **`tradingFee`**: The fee charged per trade, expressed in token units to a precision of 18 decimals.
  * **`maxLeverage`**: The maximum leverage allowed for trading this symbol.
  * **`fundingRateEpochDuration`**: The duration of each funding rate epoch, during which interest rates are recalculated.
  * **`fundingRateWindowTime`**: The window of time within an epoch when the funding rate can be changed.

**Storage Interactions**:

* Calls the `addSymbol` function for each symbol in the array, which handles all storage interactions such as updating the `SymbolStorage` with new symbol IDs and their respective parameters.

### setSymbolFundingState()

**Description**: Adjusts the funding rate parameters for a specific trading symbol within the system.

```solidity
function setSymbolFundingState(
    uint256 symbolId,
    uint256 fundingRateEpochDuration,
    uint256 fundingRateWindowTime
) external onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
    require(symbolId >= 1 && symbolId <= symbolLayout.lastId, "ControlFacet: Invalid id");
    require(
        fundingRateWindowTime < fundingRateEpochDuration / 2,
        "ControlFacet: High window time"
    );
    symbolLayout.symbols[symbolId].fundingRateEpochDuration = fundingRateEpochDuration;
    symbolLayout.symbols[symbolId].fundingRateWindowTime = fundingRateWindowTime;
    emit SetSymbolFundingState(symbolId, fundingRateEpochDuration, fundingRateWindowTime);
}
```

**Input Parameters**:

* **`symbolId`**: The unique identifier of the symbol for which the funding state is being set.
* **`fundingRateEpochDuration`**: The total duration of the funding rate epoch.
* **`fundingRateWindowTime`**: The window of time within an epoch when the funding rate can be changed.

**Storage Interactions**:

Updates the `fundingRateEpochDuration` and `fundingRateWindowTime` for the specified symbol in `SymbolStorage`, ensuring that the symbol's financial operations adhere to the newly defined timings.

**Event Structure**:

```solidity
event SetSymbolFundingState(uint256 symbolId, uint256 fundingRateEpochDuration, uint256 fundingRateWindowTime);
```

### setSymbolMaxLeverage()

**Description**: Adjusts the maximum leverage allowed for a specific trading symbol within the platform.&#x20;

```solidity
function setSymbolMaxLeverage(
    uint256 symbolId,
    uint256 maxLeverage
) external onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
    require(symbolId >= 1 && symbolId <= symbolLayout.lastId, "ControlFacet: Invalid id");
    emit SetSymbolMaxLeverage(symbolId, symbolLayout.symbols[symbolId].maxLeverage, maxLeverage);
    symbolLayout.symbols[symbolId].maxLeverage = maxLeverage;
}
```

**Input Parameters**:

* **`symbolId`**: The identifier of the symbol for which the maximum leverage is being set.
* **`maxLeverage`**: The new maximum leverage value to be set for the symbol.

**Storage Interactions**:

* Updates the `maxLeverage` parameter for the specified symbol in `SymbolStorage`.

**Event Structure**:

```solidity
event SetSymbolMaxLeverage(uint256 symbolId, uint256 previousMaxLeverage, uint256 newMaxLeverage);
```

### setSymbolAcceptableValues()

**Description**: Updates the minimum acceptable quote value and the minimum acceptable portion of liquidator fees (LF) for a specific trading symbol.&#x20;

```solidity
function setSymbolAcceptableValues(
    uint256 symbolId,
    uint256 minAcceptableQuoteValue,
    uint256 minAcceptablePortionLF
) external onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
    require(symbolId >= 1 && symbolId <= symbolLayout.lastId, "ControlFacet: Invalid id");
    emit SetSymbolAcceptableValues(
        symbolId,
        symbolLayout.symbols[symbolId].minAcceptableQuoteValue,
        symbolLayout.symbols[symbolId].minAcceptablePortionLF,
        minAcceptableQuoteValue,
        minAcceptablePortionLF
    );
    symbolLayout.symbols[symbolId].minAcceptableQuoteValue = minAcceptableQuoteValue;
    symbolLayout.symbols[symbolId].minAcceptablePortionLF = minAcceptablePortionLF;
}
```

**Input Parameters**:

* **`symbolId`**: The identifier of the symbol for which the values are being set.
* **`minAcceptableQuoteValue`**: The new minimum quote value that must be met for trading transactions involving this symbol to be considered valid.
* **`minAcceptablePortionLF`**: The new minimum liquidator fee portion awarded to the liquidator for transactions involving this symbol.

**Storage Interactions**:

* Verifies the validity of the `symbolId` within the range of registered symbols in `SymbolStorage`.
* Updates the `minAcceptableQuoteValue` and `minAcceptablePortionLF` for the specified symbol in `SymbolStorage`, ensuring that trading parameters align with the platform's financial and risk management policies.

**Event Structure**:

```solidity
event SetSymbolAcceptableValues(
    uint256 symbolId,
    uint256 previousMinAcceptableQuoteValue,
    uint256 previousMinAcceptablePortionLF,
    uint256 newMinAcceptableQuoteValue,
    uint256 newMinAcceptablePortionLF
);
```

### setSymbolTradingFee()

**Description**: Adjusts the trading fee for a specific trading symbol within the system.&#x20;

```solidity
function setSymbolTradingFee(
    uint256 symbolId,
    uint256 tradingFee
) external onlyRole(LibAccessibility.SYMBOL_MANAGER_ROLE) {
    SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
    require(symbolId >= 1 && symbolId <= symbolLayout.lastId, "ControlFacet: Invalid id");
    emit SetSymbolTradingFee(symbolId, symbolLayout.symbols[symbolId].tradingFee, tradingFee);
    symbolLayout.symbols[symbolId].tradingFee = tradingFee;
}
```

**Input Parameters**:

* **`symbolId`**: The identifier of the symbol for which the trading fee is being set.
* **`tradingFee`**: The new trading fee amount.

**Storage Interactions**:

* Validates the existence of the symbol by checking if the `symbolId` is within the valid range of registered symbols in `SymbolStorage`.
* Updates the `tradingFee` for the specified symbol in `SymbolStorage`.

**Event Structure**:

```solidity
event SetSymbolTradingFee(uint256 symbolId, uint256 previousTradingFee, uint256 newTradingFee);
```

### setDeallocateCooldown()

**Description**: Sets the cooldown period for deallocation actions within the system.&#x20;

```solidity
function setDeallocateCooldown(
    uint256 deallocateCooldown
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetDeallocateCooldown(MAStorage.layout().deallocateCooldown, deallocateCooldown);
    MAStorage.layout().deallocateCooldown = deallocateCooldown;
}
```

**Input Parameters**:

* **`deallocateCooldown`**: The duration (in seconds) to set as the new cooldown period for deallocation actions.

**Storage Interactions**:

* Updates the `deallocateCooldown` in `MAStorage`.

**Event Structure**:

```solidity
event SetDeallocateCooldown(uint256 previousCooldown
```

### setForceCancelCooldown()

**Description**: Sets the cooldown period for force cancellation actions.&#x20;

```solidity
function setForceCancelCooldown(
    uint256 forceCancelCooldown
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetForceCancelCooldown(MAStorage.layout().forceCancelCooldown, forceCancelCooldown);
    MAStorage.layout().forceCancelCooldown = forceCancelCooldown;
}
```

**Input Parameters**:

* **`forceCancelCooldown`**: The duration (in seconds) to set as the new cooldown period for force cancellation actions.

**Storage Interactions**:

* Updates the `forceCancelCooldown` in `MAStorage`. This modification directly influences the minimum time interval between requesting a cancel and performing a force cancel action.

**Event Structure**:

```solidity
event SetForceCancelCooldown(uint256 previousCooldown, uint256 newCooldown);
```

### setForceCloseCooldown()

**Description**: Adjusts the cooldown period for force close action.&#x20;

```solidity
function setForceCloseCooldown(
    uint256 forceCloseCooldown
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetForceCloseCooldown(MAStorage.layout().forceCloseCooldown, forceCloseCooldown);
    MAStorage.layout().forceCloseCooldown = forceCloseCooldown;
}
```

**Input Parameters**:

* **`forceCloseCooldown`**: The duration (in seconds) to set as the new cooldown period for force close actions.

**Storage Interactions**:

* Updates the `forceCloseCooldown` in `MAStorage`. This modification directly influences the minimum time interval between requesting a close and performing a force close action.

**Event Structure**:

```solidity
event SetForceCloseCooldown(uint256 previousCooldown, uint256 newCooldown);
```

### setForceCancelCloseCooldown()

**Description**: Sets the cooldown period for the force cancellation of close actions within the system.&#x20;

```solidity
function setForceCancelCloseCooldown(
    uint256 forceCancelCloseCooldown
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetForceCancelCloseCooldown(
        MAStorage.layout().forceCancelCloseCooldown,
        forceCancelCloseCooldown
    );
    MAStorage.layout().forceCancelCloseCooldown = forceCancelCloseCooldown;
}
```

**Input Parameters**:

* **`forceCancelCloseCooldown`**: The duration (in seconds) to set as the new cooldown period for cancelling force close actions.

**Storage Interactions**:

* Updates the `forceCancelCloseCooldown` in `MAStorage`.&#x20;

**Event Structure**:

```solidity
event SetForceCancelCloseCooldown(uint256 previousCooldown, uint256 newCooldown);
```

### setLiquidatorShare()

**Description**: Adjusts the share percentage that a liquidator receives when executing a liquidation action.&#x20;

```solidity
function setLiquidatorShare(
    uint256 liquidatorShare
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetLiquidatorShare(MAStorage.layout().liquidatorShare, liquidatorShare);
    MAStorage.layout().liquidatorShare = liquidatorShare;
}
```

**Input Parameters**:

* **`liquidatorShare`**: The new percentage (expressed as a fraction of 1e18 for precision) that a liquidator will receive as a reward for executing a liquidation.

**Storage Interactions**:

* Updates the `liquidatorShare` in `MAStorage`.

**Event Structure**:

```solidity
event SetLiquidatorShare(uint256 previousShare, uint256 newShare);
```

### setForceCloseGapRatio()

**Description**: Configures the ratio used to determine the gap between the market price and the trigger price for force-close actions.&#x20;

```solidity
function setForceCloseGapRatio(
    uint256 forceCloseGapRatio
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetForceCloseGapRatio(MAStorage.layout().forceCloseGapRatio, forceCloseGapRatio);
    MAStorage.layout().forceCloseGapRatio = forceCloseGapRatio;
}
```

**Input Parameters**:

* **`forceCloseGapRatio`**: The new gap ratio applied to the requestedClosePrice which allows a user to force close a position.&#x20;

**Storage Interactions**:

* Updates the `forceCloseGapRatio` in `MAStorage`.&#x20;

**Event Structure**:

```solidity
event SetForceCloseGapRatio(uint256 previousRatio, uint256 newRatio);
```

### setPendingQuotesValidLength()

**Description**: Sets the maximum amount of valid pending quotes for a partyA.

```solidity
function setPendingQuotesValidLength(
    uint256 pendingQuotesValidLength
) external onlyRole(LibAccessibility.SETTER_ROLE) {
    emit SetPendingQuotesValidLength(
        MAStorage.layout().pendingQuotesValidLength,
        pendingQuotesValidLength
    );
    MAStorage.layout().pendingQuotesValidLength = pendingQuotesValidLength;
}
```

**Input Parameters**:

* **`pendingQuotesValidLength`**: The maximum amount of pending quotes at one time for partyA.

**Storage Interactions**:

* Updates the `pendingQuotesValidLength` in `MAStorage`. This adjustment ensures that the system accurately reflects the operational window for executing trades based on received quotes, adapting to market dynamics or regulatory requirements.

**Event Structure**:

```solidity
event SetPendingQuotesValidLength(uint256 previousLength, uint256 newLength);
```

## Pause/Miscellaneous functions

### setFeeCollector()

```solidity
    function setFeeCollector(
        address feeCollector
    ) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        require(feeCollector != address(0),"ControlFacet: Zero address");
        emit SetFeeCollector(GlobalAppStorage.layout().feeCollector, feeCollector);
        GlobalAppStorage.layout().feeCollector = feeCollector;
    }

```

**Purpose**: Assigns the `feeCollector` address responsible for receiving system fees. Ensures the address is non-zero and updates `GlobalAppStorage`.

### pauseGlobal()

```solidity
function pauseGlobal() external onlyRole(LibAccessibility.PAUSER_ROLE) {
        GlobalAppStorage.layout().globalPaused = true;
        emit PauseGlobal();
    }
```

**Purpose**: Pauses all global operations within the system. Sets `globalPaused` to true in `GlobalAppStorage`.

### pauseLiquidation()

```solidity
function pauseLiquidation() external onlyRole(LibAccessibility.PAUSER_ROLE) {
        GlobalAppStorage.layout().liquidationPaused = true;
        emit PauseLiquidation();
    }
```

**Purpose**: Temporarily halts all liquidation processes. Activates the `liquidationPaused` state in `GlobalAppStorage`.

### pauseAccounting()

```solidity
    function pauseAccounting() external onlyRole(LibAccessibility.PAUSER_ROLE) {
        GlobalAppStorage.layout().accountingPaused = true;
        emit PauseAccounting();
    }
```

**Purpose**: Suspends all accounting activities, setting `accountingPaused` to true in `GlobalAppStorage`.

### pausePartyAActions()

```solidity
    function pausePartyAActions() external onlyRole(LibAccessibility.PAUSER_ROLE) {
        GlobalAppStorage.layout().partyAActionsPaused = true;
        emit PausePartyAActions();
    }
```

**Purpose**: Freezes all actions by Party A, setting `partyAActionsPaused` to true in `GlobalAppStorage`.

### pausePartyBActions()

```solidity
    function pausePartyBActions() external onlyRole(LibAccessibility.PAUSER_ROLE) {
        GlobalAppStorage.layout().partyBActionsPaused = true;
        emit PausePartyBActions();
    }

```

**Purpose**: Disables all operational actions for Party B, setting `partyBActionsPaused` to true in `GlobalAppStorage`.

### activeEmergencyMode()

```solidity
    function activeEmergencyMode() external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        GlobalAppStorage.layout().emergencyMode = true;
        emit ActiveEmergencyMode();
    }
```

**Purpose**: Activates emergency mode across the system, setting `emergencyMode` to true in `GlobalAppStorage`.

### unpauseGlobal()

```solidity
function unpauseGlobal() external onlyRole(LibAccessibility.UNPAUSER_ROLE) {
        GlobalAppStorage.layout().globalPaused = false;
        emit UnpauseGlobal();
    }
```

**Purpose**: Resumes all previously paused global operations by setting `globalPaused` to false in `GlobalAppStorage`.

### unpauseLiquidation()

```solidity
    function unpauseLiquidation() external onlyRole(LibAccessibility.UNPAUSER_ROLE) {
        GlobalAppStorage.layout().liquidationPaused = false;
        emit UnpauseLiquidation();
    }

```

**Purpose**: Reactivates liquidation processes by setting `liquidationPaused` to false in `GlobalAppStorage`.

### unpauseAccounting()

```solidity
    function unpauseAccounting() external onlyRole(LibAccessibility.UNPAUSER_ROLE) {
        GlobalAppStorage.layout().accountingPaused = false;
        emit UnpauseAccounting();
    }

```

**Purpose**: Resumes all accounting activities by setting `accountingPaused` to false in `GlobalAppStorage`.

### unpausePartyAActions()

```solidity
    function unpausePartyAActions() external onlyRole(LibAccessibility.UNPAUSER_ROLE) {
        GlobalAppStorage.layout().partyAActionsPaused = false;
        emit UnpausePartyAActions();
    }

```

**Purpose**: Allows Party A to resume normal operations by setting `partyAActionsPaused` to false in `GlobalAppStorage`.

### unpausePartyBActions()

```solidity
    function unpausePartyBActions() external onlyRole(LibAccessibility.UNPAUSER_ROLE) {
        GlobalAppStorage.layout().partyBActionsPaused = false;
        emit UnpausePartyBActions();
    }

```

**Purpose**: Re-enables Party B actions by setting `partyBActionsPaused` to false in `GlobalAppStorage`.

### setLiquidationTimeout()

```solidity
    function setLiquidationTimeout(
        uint256 liquidationTimeout
    ) external onlyRole(LibAccessibility.SETTER_ROLE) {
        emit SetLiquidationTimeout(MAStorage.layout().liquidationTimeout, liquidationTimeout);
        MAStorage.layout().liquidationTimeout = liquidationTimeout;
    }

```

**Purpose**: Sets the timeout period for liquidations, updating the `liquidationTimeout` in `MAStorage`.

### suspendedAddress()

```solidity
    function suspendedAddress(
        address user
    ) external onlyRole(LibAccessibility.SUSPENDER_ROLE) {
        require(user != address(0),"ControlFacet: Zero address");
        emit SetSuspendedAddress(user, true);
        AccountStorage.layout().suspendedAddresses[user] = true;
    }

```

**Purpose**: Suspends activities for a specified user address, ensuring it is non-zero and updating `suspendedAddresses` in `AccountStorage`.

### unsuspendedAddress()

```solidity
    function unsuspendedAddress(
        address user
    ) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        require(user != address(0),"ControlFacet: Zero address");
        emit SetSuspendedAddress(user, false);
        AccountStorage.layout().suspendedAddresses[user] = false;
    }

```

**Purpose**: Lifts suspension on a specified user address, ensuring it is non-zero and updating `suspendedAddresses` in `AccountStorage`.

### deactiveEmergencyMode()

```solidity
    function deactiveEmergencyMode() external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        GlobalAppStorage.layout().emergencyMode = false;
        emit DeactiveEmergencyMode();
    }
```

**Purpose**: Deactivates emergency mode across the system, updating `emergencyMode` in `GlobalAppStorage`.

### setBalanceLimitPerUser()

```solidity
    function setBalanceLimitPerUser(
        uint256 balanceLimitPerUser
    ) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        emit SetBalanceLimitPerUser(balanceLimitPerUser);
        GlobalAppStorage.layout().balanceLimitPerUser = balanceLimitPerUser;
    }

```

**Purpose**: Sets the maximum balance limit per user, updating this value in `GlobalAppStorage`.

### setPartyBEmergencyStatus()

```solidity
    function setPartyBEmergencyStatus(
        address[] memory partyBs,
        bool status
    ) external onlyRole(LibAccessibility.DEFAULT_ADMIN_ROLE) {
        for (uint8 i; i < partyBs.length; i++) {
            require(partyBs[i] != address(0),"ControlFacet: Zero address");
            GlobalAppStorage.layout().partyBEmergencyStatus[partyBs[i]] = status;
            emit SetPartyBEmergencyStatus(partyBs[i], status);
        }
    }
```

**Purpose**: Sets the emergency status for multiple Party B addresses, ensuring non-zero addresses and updating `partyBEmergencyStatus` in `GlobalAppStorage`.
