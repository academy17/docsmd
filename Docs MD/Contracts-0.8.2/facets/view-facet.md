# View Facet

This facet covers all functions related to viewing account details, symbol details, statuses etc.

## Accounts

### **balanceOf()**

**Description**: Retrieves the balance of a specified user.

```solidity
    function balanceOf(address user) external view returns (uint256) {
        return AccountStorage.layout().balances[user];
    } 
```

**Parameter**: `user` - Address of the user.

**Returns**: The current balance of the user in the system.

### **partyAStats()**

**Description**: Provides detailed statistics related to Party A's trading and liquidation status.

```solidity
    function partyAStats(
        address partyA
    )
        external
        view
        returns (
            bool,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256,
            uint256
        )
    {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        MAStorage.Layout storage maLayout = MAStorage.layout();
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        return (
            maLayout.liquidationStatus[partyA],
            accountLayout.allocatedBalances[partyA],
            accountLayout.lockedBalances[partyA].cva,
            accountLayout.lockedBalances[partyA].lf,
            accountLayout.lockedBalances[partyA].partyAmm,
            accountLayout.lockedBalances[partyA].partyBmm,
            accountLayout.pendingLockedBalances[partyA].cva,
            accountLayout.pendingLockedBalances[partyA].lf,
            accountLayout.pendingLockedBalances[partyA].partyAmm,
            accountLayout.pendingLockedBalances[partyA].partyBmm,
            quoteLayout.partyAPositionsCount[partyA],
            quoteLayout.partyAPendingQuotes[partyA].length,
            accountLayout.partyANonces[partyA],
            quoteLayout.quoteIdsOf[partyA].length
        );
    }
```

**Parameter**: `partyA` - Address of Party A.

**Returns**: A tuple containing various counts and status flags related to Party A, including liquidation status, allocated balances, locked values, and nonce counts.

### **balanceInfoOfPartyA()**

**Description**: Retrieves comprehensive balance details for Party A.

```solidity
    function balanceInfoOfPartyA(
        address partyA
    )
        external
        view
        returns (uint256, uint256, uint256, uint256, uint256, uint256, uint256, uint256, uint256)
    {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        return (
            accountLayout.allocatedBalances[partyA],
            accountLayout.lockedBalances[partyA].cva,
            accountLayout.lockedBalances[partyA].lf,
            accountLayout.lockedBalances[partyA].partyAmm,
            accountLayout.lockedBalances[partyA].partyBmm,
            accountLayout.pendingLockedBalances[partyA].cva,
            accountLayout.pendingLockedBalances[partyA].lf,
            accountLayout.pendingLockedBalances[partyA].partyAmm,
            accountLayout.pendingLockedBalances[partyA].partyBmm
        );
    }

```

**Parameter**: `partyA` - Address of Party A.

**Returns**: A tuple of balances and locked values specific to Party A.

### **balanceInfoOfPartyB()**

**Description**: Fetches balance details for Party B relative to Party A.

```solidity
    function balanceInfoOfPartyB(
        address partyB,
        address partyA
    )
        external
        view
        returns (uint256, uint256, uint256, uint256, uint256, uint256, uint256, uint256, uint256)
    {
        AccountStorage.Layout storage accountLayout = AccountStorage.layout();
        return (
            accountLayout.partyBAllocatedBalances[partyB][partyA],
            accountLayout.partyBLockedBalances[partyB][partyA].cva,
            accountLayout.partyBLockedBalances[partyB][partyA].lf,
            accountLayout.partyBLockedBalances[partyB][partyA].partyAmm,
            accountLayout.partyBLockedBalances[partyB][partyA].partyBmm,
            accountLayout.partyBPendingLockedBalances[partyB][partyA].cva,
            accountLayout.partyBPendingLockedBalances[partyB][partyA].lf,
            accountLayout.partyBPendingLockedBalances[partyB][partyA].partyAmm,
            accountLayout.partyBPendingLockedBalances[partyB][partyA].partyBmm
        );
    }

```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A they are associated with.

**Returns**: A tuple of balances and locked values related to Party B with respect to Party A.

### **allocatedBalanceOfPartyA()**

**Description**: Gets the total allocated balance for Party A.

```solidity
    function allocatedBalanceOfPartyA(address partyA) external view returns (uint256) {
        return AccountStorage.layout().allocatedBalances[partyA];
    }
```

**Parameter**: `partyA` - Address of Party A.

**Returns**: Total allocated balance for Party A.

### **allocatedBalanceOfPartyB()**

**Description**: Retrieves the total allocated balance for Party B concerning Party A.

```solidity
    function allocatedBalanceOfPartyB(
        address partyB,
        address partyA
    ) external view returns (uint256) {
        return AccountStorage.layout().partyBAllocatedBalances[partyB][partyA];
    }

```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A they are associated with.

**Returns**: Allocated balance for Party B with respect to Party A.

### **allocatedBalanceOfPartyBs()**

* **Description**: Fetches allocated balances for multiple Party Bs concerning a single Party A.

```solidity
    function allocatedBalanceOfPartyBs(
        address partyA,
        address[] memory partyBs
    ) external view returns (uint256[] memory) {
        uint256[] memory allocatedBalances = new uint256[](partyBs.length);
        for (uint256 i = 0; i < partyBs.length; i++) {
            allocatedBalances[i] = AccountStorage.layout().partyBAllocatedBalances[partyBs[i]][
                        partyA
                ];
        }
        return allocatedBalances;
    }

```

**Parameters**:

* `partyA` - Address of Party A.
* `partyBs` - Array of Party B addresses.
* **Returns**: Array of allocated balances for each Party B.

### **withdrawCooldownOf()**

**Description**: Obtains the withdrawal cooldown for a user.

```solidity
    function withdrawCooldownOf(address user) external view returns (uint256) {
        return AccountStorage.layout().withdrawCooldown[user];
    }
```

**Parameter**: `user` - Address of the user.

**Returns**: Cooldown period before the user can make a withdrawal.

### **nonceOfPartyA()**

**Description**: Provides the current nonce for Party A, which tracks the number of transactions.

```solidity
    function nonceOfPartyA(address partyA) external view returns (uint256) {
        return AccountStorage.layout().partyANonces[partyA];
    }
```

* **Parameter**: `partyA` - Address of Party A.
* **Returns**: Current nonce of Party A.

### **nonceOfPartyB()**

**Description**: Fetches the nonce for Party B specific to their transactions with Party A.

```solidity
    function nonceOfPartyB(address partyB, address partyA) external view returns (uint256) {
        return AccountStorage.layout().partyBNonces[partyB][partyA];
    }
```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.

**Returns**: Current nonce of Party B for transactions with Party A.

### **isSuspended()**

**Description**: Checks if a user's account is suspended.

```solidity
    function isSuspended(address user) external view returns (bool) {
        return AccountStorage.layout().suspendedAddresses[user];
    }
```

**Parameter**: `user` - Address of the user.

**Returns**: Boolean indicating if the account is suspended.

### **getLiquidatedStateOfPartyA()**

**Description**: Retrieves the liquidation details for Party A.

```solidity
    function getLiquidatedStateOfPartyA(
        address partyA
    ) external view returns (LiquidationDetail memory) {
        return AccountStorage.layout().liquidationDetails[partyA];
    }
```

* **Parameter**: `partyA` - Address of Party A.
* **Returns**: A `LiquidationDetail` struct containing all details related to Party A’s liquidation.

```solidity
struct LiquidationDetail {
    bytes liquidationId;
    LiquidationType liquidationType;
    int256 upnl;
    int256 totalUnrealizedLoss;
    uint256 deficit;
    uint256 liquidationFee;
    uint256 timestamp;
    uint256 involvedPartyBCounts;
    int256 partyAAccumulatedUpnl;
    bool disputed;
}
```

### **getSettlementStates()**

**Description**: Provides the settlement states for multiple Party Bs with respect to Party A.

```solidity
    function getSettlementStates(
        address partyA,
        address[] memory partyBs
    ) external view returns (SettlementState[] memory) {
        SettlementState[] memory states = new SettlementState[](partyBs.length);
        for (uint256 i = 0; i < partyBs.length; i++) {
            states[i] = AccountStorage.layout().settlementStates[partyA][partyBs[i]];
        }
        return states;
    }
```

**Parameters**:

* `partyA` - Address of Party A.
* `partyBs` - Array of Party B addresses.

**Returns**: Array of `SettlementState` structs detailing the settlement status for each Party B with Party A.

```solidity
struct SettlementState {
    int256 actualAmount; 
    int256 expectedAmount; 
    uint256 cva;
    bool pending;
}
```

## Symbols

### **getSymbol()**

**Description**: Retrieves the symbol by its `symbolId`.

```solidity
    function getSymbol(uint256 symbolId) external view returns (Symbol memory) {
        return SymbolStorage.layout().symbols[symbolId];
    }

```

* **Parameter**:
  * `symbolId` - The identifier of the symbol.
* **Returns**: A `Symbol` struct containing all relevant details of the specified symbol, such as its trading parameters and status.

```solidity
struct Symbol {
    uint256 symbolId;
    string name;
    bool isValid;
    uint256 minAcceptableQuoteValue;
    uint256 minAcceptablePortionLF;
    uint256 tradingFee;
    uint256 maxLeverage;
    uint256 fundingRateEpochDuration;
    uint256 fundingRateWindowTime;
}

```

### **getSymbols()**

**Description**: Fetches a list of symbols within a specified range.

```solidity
    function getSymbols(uint256 start, uint256 size) external view returns (Symbol[] memory) {
        SymbolStorage.Layout storage symbolLayout = SymbolStorage.layout();
        if (symbolLayout.lastId < start + size) {
            size = symbolLayout.lastId - start;
        }
        Symbol[] memory symbols = new Symbol[](size);
        for (uint256 i = start; i < start + size; i++) {
            symbols[i - start] = symbolLayout.symbols[i + 1];
        }
        return symbols;
    }
```

* **Parameters**:
  * `start` - The starting index for symbol retrieval.
  * `size` - The number of symbols to retrieve from the starting index.

**Returns**: An array of `Symbol` structs representing a sequential list of trading symbols from the specified start index.

### **symbolsByQuoteId()**

**Description**: Provides symbol details for a list of quote IDs.

```solidity
    function symbolsByQuoteId(uint256[] memory quoteIds) external view returns (Symbol[] memory) {
        Symbol[] memory symbols = new Symbol[](quoteIds.length);
        for (uint256 i = 0; i < quoteIds.length; i++) {
            symbols[i] = SymbolStorage.layout().symbols[QuoteStorage.layout().quotes[quoteIds[i]].symbolId];
        }
        return symbols;
    }
```

* **Parameter**:
  * `quoteIds` - An array of quote IDs for which symbol information is required.
* **Returns**: An array of `Symbol` structs associated with the provided quote IDs.

### **symbolNameByQuoteId()**

**Description**: Retrieves the names of symbols associated with a list of quote IDs.

```solidity
    function symbolNameByQuoteId(
        uint256[] memory quoteIds
    ) external view returns (string[] memory) {
        string[] memory symbols = new string[](quoteIds.length);
        for (uint256 i = 0; i < quoteIds.length; i++) {
            symbols[i] = SymbolStorage
                .layout()
                .symbols[QuoteStorage.layout().quotes[quoteIds[i]].symbolId]
                .name;
        }
        return symbols;
    }

```

**Parameter**:

* `quoteIds` - An array of quote IDs for which symbol names are requested.

**Returns**: An array of strings containing the names of symbols related to the provided quote IDs.

### **symbolNameById()**

**Description**: Obtains the names of symbols based on an array of symbol IDs.

```solidity
    function symbolNameById(uint256[] memory symbolIds) external view returns (string[] memory) {
        string[] memory symbols = new string[](symbolIds.length);
        for (uint256 i = 0; i < symbolIds.length; i++) {
            symbols[i] = SymbolStorage.layout().symbols[symbolIds[i]].name;
        }
        return symbols;
    }

```

**Parameter**:

* `symbolIds` - An array of symbol IDs.

**Returns**: An array of strings representing the names of the symbols corresponding to the provided IDs.

## Quote

### **getQuote()**

**Description**: Fetches a specific quote by its ID.

```solidity
    function getQuote(uint256 quoteId) external view returns (Quote memory) {
        return QuoteStorage.layout().quotes[quoteId];
    }
```

**Parameter**: `quoteId` - The unique identifier of the quote.

**Returns**: A `Quote` struct containing all the details of the specified quote.

### **getQuotesByParent()**

**Description**: Retrieves a sequence of related quotes starting from a parent quote.

```solidity
    function getQuotesByParent(
        uint256 quoteId,
        uint256 size
    ) external view returns (Quote[] memory) {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        Quote[] memory quotes = new Quote[](size);
        Quote memory quote = quoteLayout.quotes[quoteId];
        quotes[0] = quote;
        for (uint256 i = 1; i < size; i++) {
            if (quote.parentId == 0) {
                break;
            }
            quote = quoteLayout.quotes[quote.parentId];
            quotes[i] = quote;
        }
        return quotes;
    }
```

**Parameters**:

* `quoteId` - ID of the parent quote.
* `size` - Number of related quotes to retrieve.

**Returns**: An array of `Quote` structs representing the hierarchy of related quotes (for partially open positions)

### **quoteIdsOf()**

**Description**: Lists IDs of quotes associated with a specific Party A, starting from a specified index.

```solidity
    function quoteIdsOf(
        address partyA,
        uint256 start,
        uint256 size
    ) external view returns (uint256[] memory) {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        if (quoteLayout.quoteIdsOf[partyA].length < start + size) {
            size = quoteLayout.quoteIdsOf[partyA].length - start;
        }
        uint256[] memory quoteIds = new uint256[](size);
        for (uint256 i = start; i < start + size; i++) {
            quoteIds[i - start] = quoteLayout.quoteIdsOf[partyA][i];
        }
        return quoteIds;
    }
```

**Parameters**:

* `partyA` - Address of Party A.
* `start` - Starting index for retrieval.
* `size` - Number of quote IDs to retrieve.

**Returns**: An array of quote IDs belonging to Party A.

### **getQuotes()**

**Description**: Retrieves a list of quotes for Party A starting from a specified index.

```solidity
    function getQuotes(
        address partyA,
        uint256 start,
        uint256 size
    ) external view returns (Quote[] memory) {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        if (quoteLayout.quoteIdsOf[partyA].length < start + size) {
            size = quoteLayout.quoteIdsOf[partyA].length - start;
        }
        Quote[] memory quotes = new Quote[](size);
        for (uint256 i = start; i < start + size; i++) {
            quotes[i - start] = quoteLayout.quotes[quoteLayout.quoteIdsOf[partyA][i]];
        }
        return quotes;
    }

```

**Parameters**:

* `partyA` - Address of Party A.
* `start` - Starting index for retrieval.
* `size` - Number of quotes to retrieve.

**Returns**: An array of `Quote` structs associated with Party A.

### **quotesLength()**

**Description**: Provides the total number of quotes associated with a user.

```solidity
    function quotesLength(address user) external view returns (uint256) {
        return QuoteStorage.layout().quoteIdsOf[user].length;
    }

```

**Parameter**: `user` - Address of the user.

**Returns**: Total number of quotes linked to the user.

### **partyAPositionsCount()**

**Description**: Counts the number of open positions for Party A.

```solidity
    function partyAPositionsCount(address partyA) external view returns (uint256) {
        return QuoteStorage.layout().partyAPositionsCount[partyA];
    }
```

**Parameter**: `partyA` - Address of Party A.

**Returns**: Count of open positions held by Party A.

### **getPartyAOpenPositions()**

**Description**: Retrieves a list of open positions for Party A.

```solidity
    function getPartyAOpenPositions(
        address partyA,
        uint256 start,
        uint256 size
    ) external view returns (Quote[] memory) {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        if (quoteLayout.partyAOpenPositions[partyA].length < start + size) {
            size = quoteLayout.partyAOpenPositions[partyA].length - start;
        }
        Quote[] memory quotes = new Quote[](size);
        for (uint256 i = start; i < start + size; i++) {
            quotes[i - start] = quoteLayout.quotes[quoteLayout.partyAOpenPositions[partyA][i]];
        }
        return quotes;
    }

```

**Parameters**:

* `partyA` - Address of Party A.
* `start` - Starting index for retrieval.
* `size` - Number of positions to retrieve.

**Returns**: An array of `Quote` structs representing open positions for Party A.

### **getPartyBOpenPositions()**

**Description**: Fetches open positions where Party B is involved with Party A.

```solidity
    function getPartyBOpenPositions(
        address partyB,
        address partyA,
        uint256 start,
        uint256 size
    ) external view returns (Quote[] memory) {
        QuoteStorage.Layout storage quoteLayout = QuoteStorage.layout();
        if (quoteLayout.partyBOpenPositions[partyB][partyA].length < start + size) {
            size = quoteLayout.partyBOpenPositions[partyB][partyA].length - start;
        }
        Quote[] memory quotes = new Quote[](size);
        for (uint256 i = start; i < start + size; i++) {
            quotes[i - start] = quoteLayout.quotes[quoteLayout.partyBOpenPositions[partyB][partyA][i]];
        }
        return quotes;
    }

```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.
* `start` - Starting index.
* `size` - Number of positions to retrieve.

**Returns**: An array of `Quote` structs representing open positions for Party B with Party A.

### **partyBPositionsCount()**

**Description**: Counts the number of open positions where Party B is involved with Party A.

```solidity
    function partyBPositionsCount(address partyB, address partyA) external view returns (uint256) {
        return QuoteStorage.layout().partyBPositionsCount[partyB][partyA];
    }

```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.

**Returns**: Total count of open positions for Party B with Party A.

### **getPartyAPendingQuotes()**

**Description**: Lists all pending quotes initiated by Party A.

```solidity
    function getPartyAPendingQuotes(address partyA) external view returns (uint256[] memory) {
        return QuoteStorage.layout().partyAPendingQuotes[partyA];
    }
```

**Parameter**: `partyA` - Address of Party A.

**Returns**: An array of quote IDs that are pending for Party A.

### **getPartyBPendingQuotes()**

**Description**: Lists all pending quotes where Party B is involved with Party A.

```solidity
    function getPartyBPendingQuotes(
        address partyB,
        address partyA
    ) external view returns (uint256[] memory) {
        return QuoteStorage.layout().partyBPendingQuotes[partyB][partyA];
    }

```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.

**Returns**: An array of quote IDs pending for Party B with Party A.

## Roles

### **hasRole()**

**Description**: Checks if a given user has been assigned a specific role.

```solidity
    function hasRole(address user, bytes32 role) external view returns (bool) {
        return GlobalAppStorage.layout().hasRole[user][role];
    }
```

**Parameters**:

* `user` - The address of the user.
* `role` - The role identifier, represented as a `bytes32` hash.

**Returns**: `true` if the user has the specified role; otherwise, `false`.

**Usage**: This function is used to verify access permissions, ensuring that operations are performed by authorized users only.

### **getRoleHash()**

**Description**: Computes the hash of a role from its string representation, which is used as the unique identifier for role-based operations.

```solidity
    function getRoleHash(string memory str) external pure returns (bytes32) {
        return keccak256(abi.encodePacked(str));
    }
```

**Parameter**:

`str` - The string name of the role.

**Returns**: A `bytes32` hash of the role string, which is used to uniquely identify the role across the system.

## MasterAgreement

### **getCollateral()**

**Description**: Retrieves the address of the collateral token used in the trading system.

```solidity
    function getCollateral() external view returns (address) {
        return GlobalAppStorage.layout().collateral;
    }
```

**Returns**: Address of the collateral token.

### **getFeeCollector()**

**Description**: Fetches the address designated as the fee collector.

```solidity
    function getFeeCollector() external view returns (address) {
        return GlobalAppStorage.layout().feeCollector;
    }
```

**Returns**: Address where trading fees are collected.

### **isPartyALiquidated()**

**Description**: Checks if Party A has been liquidated.

```solidity
    function isPartyALiquidated(address partyA) external view returns (bool) {
        return MAStorage.layout().liquidationStatus[partyA];
    }
```

**Parameter**:

* `partyA` - Address of Party A.

**Returns**: `true` if Party A has been liquidated, otherwise `false`.

### **isPartyBLiquidated()**

**Description**: Determines whether Party B has been liquidated in relation to Party A.

```solidity
    function isPartyBLiquidated(address partyB, address partyA) external view returns (bool) {
        return MAStorage.layout().partyBLiquidationStatus[partyB][partyA];
    }
```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.

**Returns**: `true` if Party B related to Party A has been liquidated, otherwise `false`.

### **isPartyB()**

**Description**: Verifies if the user is registered as Party B.

```solidity
    function isPartyB(address user) external view returns (bool) {
        return MAStorage.layout().partyBStatus[user];
    }
```

**Parameter**:

* `user` - Address of the user to check.

**Returns**: `true` if the user is registered as Party B, otherwise `false`.

### **pendingQuotesValidLength()**

**Description**: Retrieves the maximum allowable length for pending quotes.

```solidity
    function pendingQuotesValidLength() external view returns (uint256) {
        return MAStorage.layout().pendingQuotesValidLength;
    }
```

**Returns**: Maximum number of pending quotes allowed.

### **forceCloseGapRatio()**

**Description**: Gets the gap ratio used in forced close calculations.

```solidity
    function forceCloseGapRatio() external view returns (uint256) {
        return MAStorage.layout().forceCloseGapRatio;
    }

```

**Returns**: Gap ratio value.

### **forceClosePricePenalty()**

**Description**: Provides the penalty applied to price calculations during a forced close operation.

```solidity
    function forceClosePricePenalty() external view returns (uint256) {
        return MAStorage.layout().forceClosePricePenalty;
    }
```

**Returns**: Penalty value.

### **liquidatorShare()**

**Description**: Fetches the share percentage that a liquidator receives during the liquidation process.

```solidity
    function liquidatorShare() external view returns (uint256) {
        return MAStorage.layout().liquidatorShare;
    }
```

**Returns**: Share percentage as a uint256.

### **liquidationTimeout()**

**Description**: Returns the timeout period for liquidation actions to be valid.

```solidity
    function liquidationTimeout() external view returns (uint256) {
        return MAStorage.layout().liquidationTimeout;
    }

```

**Returns**: Timeout period in seconds.

### **partyBLiquidationTimestamp()**

**Description**: Provides the timestamp when Party B's liquidation was initiated concerning Party A.

```solidity
    function partyBLiquidationTimestamp(
        address partyB,
        address partyA
    ) external view returns (uint256) {
        return MAStorage.layout().partyBLiquidationTimestamp[partyB][partyA];
    }
```

**Parameters**:

* `partyB` - Address of Party B.
* `partyA` - Address of Party A.

**Returns**: Timestamp of liquidation initiation.

### **coolDownsOfMA()**

**Description**: Retrieves cooldown periods for various operations in Master Agreements.

```solidity
    function coolDownsOfMA() external view returns (uint256, uint256, uint256, uint256) {
        return (
            MAStorage.layout().deallocateCooldown,
            MAStorage.layout().forceCancelCooldown,
            MAStorage.layout().forceCancelCloseCooldown,
            MAStorage.layout().forceCloseCooldown
        );
    }
```

**Returns**: A tuple containing cooldown periods for deallocation, force cancellation, force cancel close and force close functions.

## Misc

### **getMuonConfig()**

**Description**: Retrieves the configuration times related to Muon's signature validity for prices and unrealized profits and losses.

```solidity
    function getMuonConfig()
        external
        view
        returns (uint256 upnlValidTime, uint256 priceValidTime, uint256 priceQuantityValidTime)
    {
        upnlValidTime = MuonStorage.layout().upnlValidTime;
        priceValidTime = MuonStorage.layout().priceValidTime;
        priceQuantityValidTime = MuonStorage.layout().priceQuantityValidTime;
    }

```

**Returns**:

* `upnlValidTime` - Duration for which the unrealized profit and loss (UPNL) signature is considered valid.
* `priceValidTime` - Duration for which the price signature is considered valid.
* `priceQuantityValidTime` - Validity duration for the price quantity signature.

### **getMuonIds()**

**Description**: Fetches the identifiers and public key used for Muon's operations.

```solidity
    function getMuonIds()
        external
        view
        returns (uint256 muonAppId, PublicKey memory muonPublicKey, address validGateway)
    {
        muonAppId = MuonStorage.layout().muonAppId;
        muonPublicKey = MuonStorage.layout().muonPublicKey;
        validGateway = MuonStorage.layout().validGateway;
    }
```

**Returns**:

* `muonAppId` - Application identifier used within the Muon network.
* `muonPublicKey` - Public key associated with the application in the Muon network.
* `validGateway` - Address of the gateway used for Muon network interactions.

### **pauseState()**

**Description**: Provides the current pause states for various operational aspects of the trading system.

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
    {
        GlobalAppStorage.Layout storage appLayout = GlobalAppStorage.layout();
        return (
            appLayout.globalPaused,
            appLayout.liquidationPaused,
            appLayout.accountingPaused,
            appLayout.partyBActionsPaused,
            appLayout.partyAActionsPaused,
            appLayout.emergencyMode
        );
    }
```

**Returns**:

* `globalPaused` - Indicates if the entire system is paused.
* `liquidationPaused` - Shows if liquidation processes are currently paused.
* `accountingPaused` - Status indicating if accounting related actions are paused.
* `partyBActionsPaused` - Pause status for actions initiated by Party B.
* `partyAActionsPaused` - Pause status for actions initiated by Party A.
* `emergencyMode` - Indicates if the system is in emergency mode.

### **getPartyBEmergencyStatus()**

**Description**: Checks if Party B is in an emergency status.

```solidity
    function getPartyBEmergencyStatus(address partyB) external view returns (bool isEmergency) {
        return GlobalAppStorage.layout().partyBEmergencyStatus[partyB];
    }
```

**Parameter**:

* `partyB` - Address of Party B.

**Returns**: `true` if Party B is in an emergency status, otherwise `false`.

### **getBalanceLimitPerUser()**

**Description**: Retrieves the maximum balance limit per user.

```solidity
    function getBalanceLimitPerUser() external view returns (uint256) {
        return GlobalAppStorage.layout().balanceLimitPerUser;
    }

```

**Returns**: Balance limit per user as a uint256.

### **verifyMuonTSSAndGateway()**

**Description**: Verifies signatures and gateway authentication for Muon network transactions.

```solidity
    function verifyMuonTSSAndGateway(
        bytes32 hash,
        SchnorrSign memory sign,
        bytes memory gatewaySignature
    ) external view {
        LibMuon.verifyTSSAndGateway(hash, sign, gatewaySignature);
    }
```

**Parameters**:

* `hash` - The hash to be verified.
* `sign` - The Schnorr signature data.
* `gatewaySignature` - The signature from the gateway.

**Note**: This function is for validation and does not return a value but is essential for security checks.

### **getNextQuoteId()**

**Description**: Retrieves the next available quote identifier that will be assigned to a new quote.

```solidity
    function getNextQuoteId() external view returns (uint256){
        return QuoteStorage.layout().lastId;
    }
```

**Returns**: Next quote ID as a uint256.

## **getQuotesWithBitmap()**

The `getQuotesWithBitmap` function is a specialized retrieval method used in blockchain applications to fetch multiple quote entries efficiently from storage using a bitmap indexing technique. This method is particularly useful in environments where gas cost and data retrieval efficiency are critical, such as in the Symmio trading platform.

```solidity
    function getQuotesWithBitmap(
        Bitmap calldata bitmap,
        uint256 gasNeededForReturn
    ) external view returns (Quote[] memory quotes) {
        QuoteStorage.Layout storage qL = QuoteStorage.layout();

        quotes = new Quote[](bitmap.size);
        uint256 quoteIndex = 0;

        for (uint256 i = 0; i < bitmap.elements.length; ++i) {
            uint256 bits = bitmap.elements[i].bitmap;
            uint256 offset = bitmap.elements[i].offset;
            while (bits > 0 && gasleft() > gasNeededForReturn) {
                if ((bits & 1) > 0) {
                    quotes[quoteIndex] = qL.quotes[offset];
                    ++quoteIndex;
                }
                ++offset;
                bits >>= 1;
            }
        }
    }
```

**Parameters:**

* **`bitmap`**: A data structure that specifies which quotes to retrieve. The bitmap is composed of elements that indicate the positions of quotes in storage that are to be fetched.

{% code title="Bitmap" %}
```solidity
    struct Bitmap {
        uint256 size;
        BitmapElement[] elements;
    }
```
{% endcode %}

{% code title="BitmapElement" %}
```solidity
    struct BitmapElement {
        uint256 offset;
        uint256 bitmap;
    }

```
{% endcode %}

* **`gasNeededForReturn`**: The minimum gas remaining required to process the return of data. This helps in ensuring that the function does not run out of gas mid-execution, especially after performing potentially costly state reads.

**Returns:**

* **`quotes`**: An array of `Quote` structs retrieved based on the indices specified in the input bitmap.

**Data Structures:**

* **`Bitmap`**: Represents a collection of indices in a compact form using bits.
* **`size`**: Total number of quotes expected to be fetched.
* **`elements[]`**: An array of `BitmapElement` structures, each representing a segment of the overall bitmap.
* **`offset`**: Starting index in the quote storage for the bitmap segment.
* **`bitmap`**: A 256-bit integer where each bit represents the presence (1) or absence (0) of a quote at the corresponding index starting from `offset`.

**Functionality Overview:**

The function iterates over the array of bitmap elements, for each element, it decodes the bitmap to determine which quotes need to be fetched. It checks each bit of the bitmap; if a bit is set (`1`), the function fetches the quote from storage at the position determined by the `offset` plus the current bit's position. This process is repeated until all bits in the bitmap have been checked or until the gas remaining is less than the `gasNeededForReturn`.\
