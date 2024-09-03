# Hedger Endpoints

## Hedger API URLs

The Hedger service is available on multiple chains with the following base URLs:

* **Base Chain**: [https://base-hedger82.rasa.capital/](https://base-hedger82.rasa.capital/)
* **Blast Chain**: [https://blast-hedger.rasa.capital/](https://blast-hedger.rasa.capital/)
* **BSC Chain**: [https://alpha-hedger.rasa.capital/](https://alpha-hedger.rasa.capital/)
* **Mantle Chain**: [https://mantle-hedger.rasa.capital/](https://mantle-hedger.rasa.capital/)

### API Endpoints

The following API endpoints are available for interfacing with the Hedger service:

### **GET Requests**

* **Contract Symbols**: `/contract-symbols`
* **Open Interest**: `/open-interest`
* **Notional Cap by Symbol**: `/notional_cap/{symbol_id}`
* **Price Range by Symbol**: `/price-range/{symbol}`
* **Error Codes**: `/error_codes`
* **Specific Error Code**: `/error_codes/{error_code}`
* **Locked Parameters for a Symbol**: `/get_locked_params/{symbol}?leverage={leverage}`
* **Market Information**: `/get_market_info`
* **Unrealized Profit and Loss by Address**: `/partyA_upnl/{address}`
* **Balance Information**: `/get_balance_info/{address}/{multi_account_address}`
* **Funding Information**: `/get_funding_info`

### **POST Requests**

* **Position State**: `/position-state/{state_id}`

### Websockets

The service also supports real-time data updates through the following websocket endpoints:

* **Unrealized P\&L Updates**: `/upnl-ws`
* **Position State Updates**: `/position-state-ws3`
* **Funding Rate Updates**: `/funding-rate-ws`

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

{% hint style="info" %}
On Binance, the shorts pay longs the funding rate, so only one value is returned. In our system, positive means partyB pays partyA, for each symbol two funding rates are returned.
{% endhint %}

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
