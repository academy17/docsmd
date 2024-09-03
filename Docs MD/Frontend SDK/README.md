---
description: A step-by-step guide on connecting to our system
---

# Frontend SDK Setup Guide

## Setup Guide&#x20;

**First Execute the following command:**

```
yarn install
```

**Navigate to one of our apps folders. For example the alpha app or the nextjs app:**

```
cd apps/alpha/
```

**To set up your environment:**

1. Locate the .env.example file within the apps/alpha directory.
2. Rename .env.example to .env.
3.  Update the fields in the .env file as required. Note that each chain requires a unique&#x20;

    `HEDGER_URL` and public RPC.

For a detailed list of all deployed contracts and their corresponding Hedger URLs and deployed addresses for testing, please refer to this table:

<table data-full-width="true"><thead><tr><th>Project Name</th><th>Version</th><th>Hedger State</th><th>Chain</th><th>SYMMIO Address</th><th>Collateral Address</th><th>MultiAccount Address</th><th>PartyBAddress</th><th>PartyB API Address</th><th width="137">Main Subgraph</th><th>Analytics Subgraph</th><th>Parties Subgraph</th><th>Funding Rate Subgraph</th><th width="144">Events Subgraph</th><th>Muon</th></tr></thead><tbody><tr><td>Core</td><td>0.8.2</td><td>Active</td><td>Blast</td><td>0x3d17f073cCb9c3764F105550B0BCF9550477D266</td><td>0x4300000000000000000000000000000000000003</td><td>0xd6ee1fd75d11989e57B57AA6Fd75f558fBf02a5e</td><td>0xECbd0788bB5a72f9dFDAc1FFeAAF9B7c2B26E456</td><td>blast-hedger.rasa.capital</td><td><a href="https://api.studio.thegraph.com/query/62454/main_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/analytics_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/analytics_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/parties_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/parties_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/fundingrate_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/fundingrate_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/events_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/events_blast_8_2/version/latest</a></td><td><a href="https://muon-oracle3.rasa.capital/v1/">https://muon-oracle3.rasa.capital/v1/</a><a href="https://muon-oracle4.rasa.capital/v1/">https://muon-oracle4.rasa.capital/v1/</a></td></tr><tr><td>IntentX (Blast)</td><td>0.8.2</td><td>Active</td><td>Blast</td><td>0x3d17f073cCb9c3764F105550B0BCF9550477D266</td><td>0x4300000000000000000000000000000000000003</td><td>0x083267D20Dbe6C2b0A83Bd0E601dC2299eD99015</td><td>0xECbd0788bB5a72f9dFDAc1FFeAAF9B7c2B26E456</td><td>blast-hedger.rasa.capital</td><td><a href="https://api.studio.thegraph.com/query/62454/main_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/analytics_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/analytics_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/parties_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/parties_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/fundingrate_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/fundingrate_blast_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/events_blast_8_2/version/latest">https://api.studio.thegraph.com/query/62454/events_blast_8_2/version/latest</a></td><td><a href="https://muon-oracle3.rasa.capital/v1/">https://muon-oracle3.rasa.capital/v1/</a><a href="https://muon-oracle4.rasa.capital/v1/">https://muon-oracle4.rasa.capital/v1/</a></td></tr><tr><td>Thena V3</td><td>0.8.2</td><td>Active</td><td>BNB</td><td>0x9A9F48888600FC9c05f11E03Eab575EBB2Fc2c8f</td><td>0x55d398326f99059ff775485246999027b3197955</td><td>0x650a2D6C263A93cFF5EdD41f836ce832F05A1cF3</td><td>0x9fa01a45E245015fA685F21763e60C60832Ed2D6</td><td></td><td><a href="https://api.studio.thegraph.com/query/62454/main_bnb_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_bnb_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/main_bnb_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_bnb_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/parties_bnb_8_2/version/latest">https://api.studio.thegraph.com/query/62454/parties_bnb_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/fundingrate_bnb_8_2/version/latest">https://api.studio.thegraph.com/query/62454/fundingrate_bnb_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/events_bnb_8_2/version/latest">https://api.studio.thegraph.com/query/62454/events_bnb_8_2/version/latest</a></td><td><a href="https://muon-oracle3.rasa.capital/v1/">https://muon-oracle3.rasa.capital/v1/</a><a href="https://muon-oracle4.rasa.capital/v1/">https://muon-oracle4.rasa.capital/v1/</a></td></tr><tr><td>IntentX</td><td>0.8.2</td><td>Active</td><td>Base</td><td>0x91Cf2D8Ed503EC52768999aA6D8DBeA6e52dbe43</td><td>0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913</td><td>0x8Ab178C07184ffD44F0ADfF4eA2ce6cFc33F3b86</td><td>0x9206D9d8F7F1B212A4183827D20De32AF3A23c59</td><td></td><td><a href="https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/parties_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/parties_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/fundingrate_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/fundingrate_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/events_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/events_base_8_2/version/latest</a></td><td><a href="https://muon-oracle2.rasa.capital/v1/">https://muon-oracle2.rasa.capital/v1/</a></td></tr><tr><td>Based</td><td>0.8.2</td><td>Active</td><td>Base</td><td>0x91Cf2D8Ed503EC52768999aA6D8DBeA6e52dbe43</td><td>0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913</td><td>0x1c03B6480a4efC2d4123ba90d7857f0e1878B780</td><td>0x9206D9d8F7F1B212A4183827D20De32AF3A23c59</td><td>base-hedger82.rasa.capital</td><td><a href="https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/main_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/parties_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/parties_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/fundingrate_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/fundingrate_base_8_2/version/latest</a></td><td><a href="https://api.studio.thegraph.com/query/62454/events_base_8_2/version/latest">https://api.studio.thegraph.com/query/62454/events_base_8_2/version/latest</a></td><td><a href="https://muon-oracle2.rasa.capital/v1/">https://muon-oracle2.rasa.capital/v1/</a></td></tr><tr><td>CloverField (Polygon)</td><td>0.8.3</td><td>Pending</td><td>Polygon</td><td>0x976c87Cd3eB2DE462Db249cCA711E4C89154537b</td><td>0x50E88C692B137B8a51b6017026Ef414651e0d5ba</td><td>0xffE2C25404525D2D4351D75177B92F18D9DaF4Af</td><td>0x5044238ea045585C704dC2C6387D66d29eD56648</td><td>polygon-hedger-test.rasa.capital</td><td><a href="https://api.thegraph.com/subgraphs/name/symmiograph/symmiomain_polygon_8_2">https://api.thegraph.com/subgraphs/name/symmiograph/symmiomain_polygon_8_2</a></td><td><a href="https://api.thegraph.com/subgraphs/name/symmiograph/symmioanalytics_polygon_8_2">https://api.thegraph.com/subgraphs/name/symmiograph/symmioanalytics_polygon_8_2</a></td><td><a href="https://api.thegraph.com/subgraphs/name/symmiograph/symmioparties_polygon_8_2">https://api.thegraph.com/subgraphs/name/symmiograph/symmioparties_polygon_8_2</a></td><td></td><td></td><td><a href="https://muon-oracle2.rasa.capital/v1/">https://muon-oracle2.rasa.capital/v1/</a><a href="https://muon-oracle3.rasa.capital/v1/">https://muon-oracle3.rasa.capital/v1/</a><a href="https://muon-oracle4.rasa.capital/v1/">https://muon-oracle4.rasa.capital/v1/</a></td></tr></tbody></table>

**Finally, execute the following command:**

```
yarn dev
```

The development server will start at `http://localhost:3000`&#x20;

**NOTE:** The Alpha and the Next.js application are identical, they just have different UIs.

## Configuring the app

#### User Configuration

Supported chains are declared in the core package's `src/constants/chains.ts`:

```typescript
export enum SupportedChainId {
  NOT_SET = 0,
  MAINNET = 1,
  ROPSTEN = 3,
  RINKEBY = 4,
  BSC = 56,
  BASE = 8453,
  BSC_TESTNET = 97,
  POLYGON = 137,
  FANTOM = 250,
  ARBITRUM = 42161,
}
```

To add a chain for the user to connect to, you should select one or multiple chains from the supported chain IDs and add it to the ClientChain variable in the `nextjs/constants/chains.ts` file.

```typescript
export const ClientChain = [
  SupportedChainId.POLYGON,
  SupportedChainId.BSC,
  SupportedChainId.BASE,
]; //Adding chains...
```

#### Configuring the Hedger API

Navigate to the `app/constants/chains/hedgers.ts` and add the chain to the `HedgerInfo` ,here's an example declaration:

```typescript
  [SupportedChainId.POLYGON]: [
    {
      apiUrl: "https://fapi.binance.com/",
      webSocketUrl: "wss://fstream.binance.com/stream",
      baseUrl: `https://${process.env.NEXT_PUBLIC_POLYGON_HEDGER_URL}`,
      webSocketUpnlUrl: `wss://${process.env.NEXT_PUBLIC_POLYGON_HEDGER_URL}/ws/upnl-ws`,
      webSocketNotificationUrl: `wss://${process.env.NEXT_PUBLIC_POLYGON_HEDGER_URL}/ws/position-state-ws3`,
      webSocketFundingRateUrl: `wss://${process.env.NEXT_PUBLIC_POLYGON_HEDGER_URL}/ws/funding-rate-ws`,
      defaultMarketId: 1,
      markets: [],
      openInterest: { total: 0, used: 0 } as OpenInterest,
      id: "",
      fetchData: true,
      clientName: "",
    },
  ],
```

The `apiUrl` and `webSocketUrl` won't change regardless of the chain because frontends listen directly to the Binance API for fetching price data.

#### Configuring Addresses

To ensure the frontend interacts properly with the deployed contracts, you'll need to update the `constants/chains/addresses.ts` file with the appropriate contract addresses for each chain. Additionally, make sure to add any necessary contract ABIs to the `/abi` folder.&#x20;

```typescript
export const CustomChain: ChainType = {
  COLLATERAL_SYMBOL: "",
  COLLATERAL_DECIMALS: ,
  COLLATERAL_ADDRESS: "",

  DIAMOND_ADDRESS: "",
  MULTI_ACCOUNT_ADDRESS: "",
  PARTY_B_WHITELIST: "",
  SIGNATURE_STORE_ADDRESS: "",

  MULTICALL3_ADDRESS: "",
  USDC_ADDRESS: "",
  WRAPPED_NATIVE_ADDRESS: "",
  ANALYTICS_SUBGRAPH_ADDRESS:
    "",
  ORDER_HISTORY_SUBGRAPH_ADDRESS:
    "",
};
```

#### Configuring Subgraphs

To fetch the balance history or order history of a user we need to determine the appropriate GraphQL subgraph URI based on the network chain ID. Then use the Apollo Client to execute a query to the subgraph. To define these you’ll have to navigate to `packages/core/src/apollo/client` and modify the `balanceHistory.ts` and `orderHistory.ts` files to reflect the new chain. For example:

```typescript
const testClient = createApolloClient(
  `${getSubgraphName(SupportedChainId.TEST)}`
);
```

Then add it as a case for the `getBalanceHistoryApolloClient` function and the `getOrderHistoryApolloClient` functions respectively:

```typescript
export function getBalanceHistoryApolloClient(chainId: SupportedChainId) {
  switch (chainId) {
    case SupportedChainId.TEST:
      return testClient;
```

#### Configuring Muon&#x20;

Muon is used in our system for verifying the price of an asset and the uPnL of a user. Muon signatures are required by the smart contracts to execute functions related to user positions.

In most scenarios, you will not need to alter the Muon base URLs, however this can be updated from  `/constants/chains/misc.ts`.&#x20;

## MultiAccount

Frontends support creating and switching between sub-accounts via the `MultiAccount` contract.

When a user interacts with the SYMM contracts through a sub-account to send or modify quotes, they'll use the `_call` function on the `MultiAccount` contract with the `account` (the address of the sub-account) and `_callData`, a bytes array. This call is forwarded to the sub-account contract through an `innerCall`, which then calls the function on the diamond.

&#x20;Users can select the account they desire from a drop-down menu in the header, allowing users to isolate positions across accounts, since all positions on a single sub-account are in **CROSS**. Users are required to provide a name as an input parameter when they create a sub-account. The hook `useActiveAccountAddress` returns the subaccount address, and the `useActiveWagmi` returns the wallet address.

## Querying the Hedger

To optimize user experience, facilitate experimentation, and aid users in identifying the most suitable hedger for their needs, SYMMIO provides a suite of off-chain information. This information is primarily classified into two categories: APIs and Web-Sockets.

It's important to emphasize that these APIs aim to reduce user request failures. They are often subject to the hedger's policies. For example, if a hedger doesn't mandate a whitelist, then the related APIs become superfluous.

Many of the API endpoints require a `MultiAccount` address as a parameter. This address refers to the address which users will interact with on the frontend for deploying sub-accounts and calling functions related to the opening and closing of positions on the SYMMIO Diamond Contract.&#x20;

## Hedger API

The Hedger API provides a suite of functionalities for users to help users monitor their positions. Querying the API offers valuable insights into the status of positions, total open interest, notional caps, and the capital requirements needed to initiate positions.

Some useful API endpoints are listed below:

### \[GET] contract-symbols

This endpoint gets the total number of symbols the hedger is offering, and returns important data pertaining to each.

**Returns:**

* **`count`**: The total number of trading symbols available through the hedger.
* **`symbols`**: An array of objects, each representing a specific trading symbol or contract offered by the hedger. The array includes detailed information about each symbol, as outlined in the fields within each object.
  * **`name`**: The name or description of the trading pair (e.g. BTCUSDT).
  * **`symbol`**: The ticker that identifies the former part of the trading pair (e.g. BTC).
  * **`asset`**: The ticker that identifies the latter part of the trading pair (e.g. USDT).
  * **`symbol_id`**: A unique integer identifier for the symbol within the hedger's system.
  * **`price_precision`**: Refers to the number of decimal places to which the price of an asset can be specified.
  * **`quantity_precision`**: Refers to the number of decimal places to which the quantity of the asset being traded can be specified.
  * **`is_valid`**: A boolean indicating whether the symbol is currently valid for trading.
  * **`min_acceptable_quote_value`**: The minimum quote value (of collateral tokens) accepted for a trade with this symbol.
  * **`min_acceptable_portion_lf`**: The minimum portion that is awarded to the liquidator on liquidation.
  * **`trading_fee`**: The fee rate charged for trading this symbol.
  * **`max_leverage`**: The maximum leverage available for trading.
  * **`max_notional_value`**: The maximum notional value of a position.
  * **`rfq_allowed`**: A boolean indicating whether Request for Quote (RFQ) is allowed.
  * **`max_funding_rate`**: The maximum funding rate allowed.
  * **`hedger_fee_open`**: The hedger fee for opening a position with this symbol.
  * **`hedger_fee_close`**: The hedger fee for closing a position with this symbol.

**Output Schema:**

```json
{
  "count": int,
  "symbols": [
    {
      "name": "",
      "symbol": "",
      "asset": "",
      "symbol_id": int,
      "price_precision": int,
      "quantity_precision": int,
      "is_valid": bool,
      "min_acceptable_quote_value": int,
      "min_acceptable_portion_lf": float,
      "trading_fee": float,
      "max_leverage": int,
      "max_notional_value": int,
      "rfq_allowed": bool,
      "max_funding_rate": int,
      "hedger_fee_open": "",
      "hedger_fee_close": ""
    },
  ]
}
```

### \[GET] open-interest

This endpoint shows the cumulative OI available to hedgers, encompassing all trading symbols. It also provides information on the used capacity.

```
api/open-interest/
```

**Returns:**

* **`total_cap`**: The total open interest cap.
* **`used`**: How much is currently in use.

**Output Schema:**

<pre class="language-json"><code class="lang-json"><strong>{ 
</strong><strong>total_cap: float, 
</strong><strong>used: float 
</strong><strong>}
</strong></code></pre>

The `getOpenInterest` function for this query can be found in the core package at `src/state/hedger/thunks.ts`.

### \[GET] notional\_cap

This endpoint shows the notional cap of a symbol provided by the hedger which represents the value of the outstanding positions that haven't yet been settled for that symbol.

<pre><code><strong>api/notional_cap/${symbolId}/
</strong></code></pre>

**Parameters:**

* **`symbolId`:** The symbol ID of the pair (eg. for `BTCUSDT`, the `symbolId` = 1).

**Returns:**

* **`total_cap`**: The total open interest cap.
* **`used`**: How much is currently in use.

**Output Schema:**

```json
{ 
total_cap: float, 
used: float 
}
```

### \[GET] price-range

This endpoint provides the maximum and minimum prices at which `partyA` can place a limit order for a specific trading pair, ensuring execution by the hedger.

```
api/price-range/${symbol}
```

**Parameters:**

* **`symbol`**: The symbol (e.g. BTCUSDT).

**Returns:**

* **`min_price`**: The minimum price for limit orders.
* **`max_price`**: The maximum price for limit orders.

**Output Schema:**

```json
{ 
min_price: float,
max_price: float
}
```

### \[GET] error\_codes

To help understand the meaning of the error codes returned from querying a position state, this API fetches a list of potential error labels and their associated error codes that may arise while interacting with the hedger system. Leaving the `error_code` blank returns all error codes.

```javascript
api/error_codes/{error_code}
```

**Parameters:**

**`error_code`**: The error code to query.

**Returns:**

```
{
"error_code":"reason"
}
```

### \[GET] get\_locked\_params

These parameters are important because they are used when opening the quote. This method returns important values related to collateral requirements for a specified pair based on the provided parameters.

```plaintext
api/get_locked_params/${symbol}?leverage=${leverage}
```

**Parameters:**

* **`symbol`**: The trading symbol (e.g. BTCUSDT)
* **`leverage`**: The leverage applied to the position. This will affect the calculation of collateral requirements.

**Returns:**

* **`cva`**: Credit Valuation Adjustment. Either `partyA` (the user) or `partyB` (the hedger) can get liquidated and `cva` is the penalty that the liquidated side should pay to the other.
* **`partyAmm`**: Maintenance Margin for `partyA`. The amount that is actually behind the position and is considered in liquidation status.
* **`lf`**: Liquidation Fee. This will be awarded to the Liquidator that liquidates the position.
* **`leverage`:** The leverage applied to the position.

**Output Schema:**

```json
{
  cva: '',
  partyAmm: '',
  lf: '',
  leverage: '',
  partyBmm: ''
}
```

### **\[GET] get\_market\_info**

This endpoint retrieves market information for various whitelisted trading pairs, it requires the deployed `MultiAccount` contract as an endpoint parameter.

`api/get_market_info/`

**Returns:**

* **`price`**: The current price of the symbol.
* **`price_change_percent`**: The percentage change in price over the last 24 hours.
* **`trade_volume`**: The total trading volume in USD for the last 24 hours.
* **`notional_cap`**: The notional cap of the symbol.

**Output Schema:**

```json
{
  "<Symbol>": {
    "price": float,
    "price_change_percent": float,
    "trade_volume": float,
    "notional_cap": float
    }
}
```

This query is used in the core package at `src/state/hedger/thunks.ts`. It's used mostly for displaying data on the [Market Information](https://cloverfield.exchange/markets) page.

### **\[GET] partyA\_upnl**

This endpoint returns the unrealized profits and losses for a `partyA`

`/partyA_upnl/{address}`

**Parameters:**

**`address`:** the sub-account address to query.

**Returns:**

The current uPnl for the user as a float.

### \[GET] get\_balance\_info

This API method retrieves balance information for a specified address and multi-account address. It returns detailed financial data for both parties involved, which is crucial for managing and analyzing account balances and associated parameters.

**Endpoint**

`api/get_balance_info/${address}/${multi_account_address}`

**Parameters**

* `address`: The sub-account address
* `multi_account_address`: The multi-account address.

**Returns**

* **party\_a**: Information for party A (the user).
  * `upnl`: Unrealized Profit and Loss .
  * `notional`: Notional value .
  * `timestamp`: Timestamp of the data.
  * `available_balance`: Available balance.
  * `allocated_balance`: Allocated balance.
  * `cva`: Credit Valuation Adjustment.
  * `lf`: Liquidation Fee.
  * `party_a_mm`: Maintenance Margin for party A.
  * `party_b_mm`: Maintenance Margin for party B.
  * `pending_cva`: Pending Credit Valuation Adjustment.
  * `pending_lf`: Pending Liquidation Fee.
  * `pending_party_a_mm`: Pending Maintenance Margin for party A.
  * `pending_party_b_mm`: Pending Maintenance Margin for party B.
  * `address`: Address of party A.
* **party\_b**: Information for party B (the counterparty).
  * `upnl`: Unrealized Profit and Loss.
  * `notional`: Notional value.
  * `timestamp`: Timestamp of the data .
  * `available_balance`: Available balance.
  * `allocated_balance`: Allocated balance.
  * `cva`: Credit Valuation Adjustment.
  * `lf`: Liquidation Fee.
  * `party_a_mm`: Maintenance Margin for party A.
  * `party_b_mm`: Maintenance Margin for party B.
  * `pending_cva`: Pending Credit Valuation Adjustment .
  * `pending_lf`: Pending Liquidation Fee.
  * `pending_party_a_mm`: Pending Maintenance Margin for party A
  * `pending_party_b_mm`: Pending Maintenance Margin for party B.
  * `address`: Address of party B.

### \[GET] get\_funding\_info

This API method retrieves funding information for various trading pairs. It returns details about the next funding time and the funding rates for both short and long positions for each specified trading pair.

`api/get_funding_info/`

**Parameters**

This endpoint does not require any parameters to be passed in the URL.

**Returns**

The response contains funding information for multiple trading pairs. Each trading pair includes the following details:

* `next_funding_time`: The timestamp of the next funding time.
* `next_funding_rate_short`: The next funding rate for short positions.
* `next_funding_rate_long`: The next funding rate for long positions.

**Output Schema:**

```javascript
  "AEVOUSDT": {
    "next_funding_time": int,
    "next_funding_rate_short": "",
    "next_funding_rate_long": ""
  },
```

### **\[POST] position-state**

This endpoint returns information regarding the state of a position, indicating if there has been any internal issue with the quote processing by returning an `action_status`. It is useful for discerning any errors associated with a specific trading position.

**Endpoint:**\
`api/position-state/${start}/${size}`

**Parameters:**

* `start` and `size` are utilized for pagination.

**Payload:**\
The payload for this POST request should be empty.

**Returns:**

* **`create_time`**: Unix timestamp indicating when the position was created.
* **`modify_time`**: Unix timestamp indicating the last modification time of the position.
* **`counterparty_address`**: The blockchain address of the counterparty involved in the position (this will be `partyA`)
* **`filled_amount_close`**: The filled amount for closing the position, or `null` if the position has not been closed.
* **`action_status`**: Indicates the success or failure status of the action, such as `success` for successful processing.
* **`error_code`**: An error code if there was an issue with processing, otherwise `null`.
* **`state_type`**: The type of state the position is in.
* **`id`**: A unique identifier for the position.
* **`quote_id`**: The identifier for the associated quote.
* **`filled_amount_open`**: The filled amount for opening the position.
* **`last_seen_action`**: Describes the last action taken on the position.
* **`failure_type`**: Describes the type of failure if the action was not successful, otherwise `null`.
* **`order_type`**: Numeric code representing the type of order.

**Output Schema:**

```json
{
  "create_time": int,
  "modify_time": int,
  "counterparty_address": "",
  "filled_amount_close": int|null,
  "action_status": "",
  "error_code": int|null,
  "state_type": "",
  "id": "",
  "quote_id": int,
  "filled_amount_open": int,
  "last_seen_action": "",
  "failure_type": "string|null",
  "order_type": int
}
```

This API is useful for displaying notifications related to a user's positions. It's applied in the `getNotifications` function in the core package's `src/state/notifications/thunks.ts`\\

## Websockets

You can use the various hedger WebSockets to stream real-time data from the hedger, ensuring that the frontend displays the most up-to-date information. Below are the most commonly used websockets:

### **Funding Rate WebSocket**

`api/ws/funding-rate-ws`

Streams real-time data on the funding rates for various trading symbols.

**Parameters:**

* `string[]` - An array of symbol names for which funding rate updates are requested (e.g., `["BTCUSDT", "ETHUSDT", "XRPUSDT"]`).

**Returns:**

* **`next_funding_time`**: The timestamp for when the next funding rate update will occur.
* **`next_funding_rate_short`**: The funding rate applied to short positions.
* **`next_funding_rate_long`**: The funding rate applied to long positions.

**Output Schema**

```
SYMBOL: {
    next_funding_time: int,
    next_funding_rate_short: '',
    next_funding_rate_long: ''
  },
```

The `getFundingRate` function for this implements this Websocket and can be found in the core package at `src/state/hedger/thunks.ts`.

### **Unrealized PnL WebSocket**

`api/ws/upnl-ws`

Streams real-time data on the profit and losses of a user (This may be deprecated soon).

**Parameters:**

* **`string`** Address of the sub-account.

**Returns:**

* **`upnl`**: Unrealized profit and loss for the account's current open positions.
* **`notional`**: The total notional value of the account's open positions.
* **`timestamp`**: The current timestamp.
* **`available_balance`**: `(allocated_balance + unpl - (cva + lf))`
* **`allocated_balance`**: The balance allocated to the sub account.
* **`cva`**: Represents the penalty the liquidated entity must pay to the other party.
* **`lf`**: Liquidation Fee. This will be awarded to the Liquidator that liquidates the position.
* **`party_a_mm`**: Maintenance Margin of `partyA`. The amount that is actually behind the position and is considered in liquidation status.
* **`party_b_mm`**: Maintenance Margin of `partyB`. The amount that is actually behind the position and is considered in liquidation status.
* **`pending_cva`**: The pending `cva`
* **`pending_lf`**: The pending `lf`
* **`pending_party_a_mm`**: The pending `mm` for `partyA`
* **`pending_party_b_mm`**: The pending `mm` for `partyB`

**Output Schema:**

```json
{
  upnl: '',
  notional: '',
  timestamp: int,
  available_balance: '',
  allocated_balance: '',
  cva: '',
  lf: '',
  party_a_mm: '',
  party_b_mm: '',
  pending_cva: '',
  pending_lf: '',
  pending_party_a_mm: '',
  pending_party_b_mm: ''
}
```

This query is implemented in the `AccountUpdater` function in `src/state/user/AllAccountsUpdater.tsx` .

### Notification WebSocket (Position State)

`api/ws/position-state-ws3`

This WebSocket streams real-time data on the state of trading positions. It ensures that the frontend can display the most current information regarding the state of various trading positions.

**Parameters:**

* `address[]`: An array of counterparty addresses for which position state updates are requested.

**Returns:**

* `id`: Identifier for the position state event.
* `create_time`: The timestamp when the position was created.
* `modify_time`: The timestamp when the position was last modified.
* `quote_id`: An identifier for the associated quote.
* `counterparty_address`: The address of the counterparty involved in the position.
* `filled_amount_open`: The amount filled for an open.
* `filled_amount_close`: The amount filled for a ckise.
* `last_seen_action`: The last action observed on the position.
* `action_status`: The status of the last action (e.g., success, failure).
* `failure_type`: The type of failure if the action was unsuccessful.
* `error_code`: The error code if applicable.
* `order_type`: The type of order associated with the position.
* `state_type`: The state type of the position (e.g., alert).
* `version`: The version of the position state data.

**Output Schema:**

```json
{
    "id": "",
    "create_time": int,
    "modify_time": int,
    "quote_id": int,
    "counterparty_address": "",
    "filled_amount_open": "",
    "filled_amount_close": "",
    "last_seen_action": "",
    "action_status": "",
    "failure_type": null,
    "error_code": null,
    "order_type": int,
    "state_type": "",
    "version": int
}
```

**Input Parameter:**

```json
{
  "address": ["0x0000000000000000000000000000000000000000"]
}
```
