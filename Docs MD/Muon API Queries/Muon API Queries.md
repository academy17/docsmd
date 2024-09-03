# The Muon API

## Introduction

The Muon API is designed to provide signed data for SYMM 3rd Party frontends, a platform that enables users to trade assets and perpetual contracts (perps) without the need for Know Your Customer (KYC) procedures. By utilizing their Ethereum wallet, users can participate in trading activities on Cloverfield.

Muon acts as an intermediary service, facilitating the generation and signing of data that is required for trading. This documentation will provide an overview of the app codebase, explaining its key components and functionalities.

## Overview of the Muon API

The Muon API is designed to provide various data related to parties, unrealized profit and loss (uPnl) and price data. It offers separate uPnl calculations for individual parties, namely "partyA" and "partyB," as well as combined uPnl calculations for both parties together. Additionally, the API allows fetching the prices of given quotes. The following sections provide technical documentation of each method.

## URL Query Endpoints

The Muon app can be queried using specific URLs, each corresponding to different networks:

* [**https://muon-oracle2.rasa.capital/v1/**](https://muon-oracle2.rasa.capital/v1/): Base, Mantle
* [**https://muon-oracle3.rasa.capital/v1/**](https://muon-oracle3.rasa.capital/v1/): BSC, Blast
* [**https://muon-oracle4.rasa.capital/v1/**](https://muon-oracle4.rasa.capital/v1/): BSC, Blast

### `uPnl_A`

The `uPnl_A` method in the Muon app is designed to verify and validate the unrealized profit and loss (uPnl) for `partyA` based on a set tolerance level.&#x20;

**Input Parameters**

* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `symmio`: Address of the Symmio diamond.
* `partyA`: The address of partyA.
* `nonce`: Unique nonce for the transaction to prevent replay attacks.
* `uPnl`: The verified uPnl value for partyA as obtained from the request data.
* `timestamp`: The blockchain timestamp at which the request data was generated.
* `chainId`: The chain identifier.

**Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=uPnl_A&params[partyA]=$address&params[chainId]=$chainId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

**Example Response**

<pre class="language-json"><code class="lang-json">{
    "success": true,
    "result": {
        "reqId": "",
        "app": "symmio",
        "appId": "",
        "method": "uPnl_A",
        "data": {
            "params": {
                "partyA": "",
                "chainId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": ,
            "uid": "",
            "result": {
                "chainId": "",
                "partyA": "",
                "symmio": "",
                "liquidationId": "",
                "latestBlockNumber": "",
                "uPnl": "",
                "loss": "",
                "notionalValueSum": "",
                "nonce": "",
                "quoteIds": [],
                "symbolIds": [],
                "symbolIdsPrices": [],
                "prices": [],
                "pricesMap": {
<strong>                }
</strong>            }
        }
    }
</code></pre>

### `partyA_overview`

The `partyA_overview` method in the Muon API provides a comprehensive overview of `partyA`'s trading status, including the unrealized profit and loss (uPnl), loss, notional value sum, and other relevant financial data. This method ensures detailed insight into `partyA`'s trading performance and current market position.

**Input Parameters**

* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `symmio`: Address of the Symmio diamond.
* `partyA`: The address of partyA.
* `nonce`: Unique nonce for the transaction to prevent replay attacks.
* `uPnl`: The verified uPnl value for partyA as obtained from the request data.
* `timestamp`: The blockchain timestamp at which the request data was generated.

**Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=partyA_overview&params[partyA]=$address&params[chainId]=$chainId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

**Example Response**

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "symmio",
        "appId": "",
        "method": "partyA_overview",
        "data": {
            "params": {
                "partyA": "",
                "chainId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "partyA": "",
                "symmio": "",
                "liquidationId": "",
                "latestBlockNumber": "",
                "uPnl": "",
                "loss": "",
                "notionalValueSum": "",
                "nonce": "",
                "quoteIds": [],
                "symbolIds": [],
                "symbolIdsPrices": [],
                "prices": [],
                "pricesMap": {
                }
                "signParams": []
            }
        }
    }
}
```

### `uPnl_A_withSymbolPrice`

The `uPnl_A_withSymbolPrice` method in the Muon API calculates the unrealized profit and loss (uPnl) for `partyA` and fetches the current price of a specified symbol.&#x20;

**Input Parameters**

* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symbolId`: The identifier of the trading symbol.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `chainId`: The chain identifier.
* `partyA`: The address of partyA.
* `symbolId`: The identifier of the trading symbol.
* `price`: The current price of the specified symbol.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number at the time of the query.
* Additional data included as the `result`.

#### **Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=uPnl_A_withSymbolPrice&params[partyA]=$address&params[chainId]=$chainId&params[symbolId]=$symbolId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

#### Example Response:

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "",
        "appId": "",
        "method": "",
        "data": {
            "params": {
                "partyA": "",
                "chainId": "",
                "symbolId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "partyA": "",
                "symbolId": "",
                "price": "",
                "symmio": "",
                "latestBlockNumber": "",
                "uPnl": "",
                "loss": "",
                "notionalValueSum": "",
                "nonce": "",
                "quoteIds": [],
                "symbolIds": [],
                "symbolIdsPrices": [],
                "prices": [],
                "pricesMap": {}
                "maxLeverages": {}
"signParams": []
```

### `uPnl_B`

The `uPnl_B` method within the Muon API calculates the unrealized profit and loss (uPnl) specifically for `partyB` in relation to `partyA`.&#x20;

**Input Parameters**

* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `chainId`: The chain identifier.
* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number at the time of the query.
* Additional data included as the `result`.

**Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=uPnl_B&params[partyB]=$partyB_address&params[partyA]=$partyA_address&params[chainId]=$chainId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

#### Example Response:

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "symmio",
        "appId": "",
        "method": "uPnl_B",
        "data": {
            "params": {
                "partyB": "",
                "partyA": "",
                "chainId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "partyB": "",
                "partyA": "",
                "symmio": "",
                "latestBlockNumber": "",
                "uPnl": "",
                "notionalValueSum": "",
                "nonce": "",
                "quoteIds": []
            },
        }
    }
}
```

### `uPnl`&#x20;

The `uPnl` method within the Muon API calculates the combined unrealized profit and loss (uPnl) for both `partyB` and `partyA`. This method provides a comprehensive view of the financial performance and interactions between the two parties.

**Input Parameters**

* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `chainId`: The chain identifier.
* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number at the time of the query.
* Additional data included as the `result`.

**Example API Query**

<pre class="language-bash"><code class="lang-bash"><strong>https://muon-oracle4.rasa.capital/v1/?app=symmio&#x26;method=uPnl&#x26;params[partyB]=$partyB_address&#x26;params[partyA]=$partyA_address&#x26;params[chainId]=$chainId&#x26;params[symmio]=$symmio&#x26;params[latestBlockNumber]=$latestBlockNumber
</strong></code></pre>

#### Example Response:

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "",
        "appId": "",
        "method": "uPnl",
        "data": {
            "params": {
                "partyB": "",
                "partyA": "",
                "chainId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "partyB": "",
                "partyA": "",
                "symmio": "",
                "latestBlockNumber": "",
                "uPnlB": "",
                "uPnlA": "",
                "notionalValueSumB": "",
                "notionalValueSumA": "",
                "nonceB": "",
                "nonceA": "",
                "pricesMap": {}
                "signParams": [

                }
            }
        }
    }
}
```

### uPnlWithSymbolPrice

The `uPnlWithSymbolPrice` method within the Muon API calculates the combined unrealized profit and loss (uPnl) for both `partyB` and `partyA` and fetches the current price of a specified symbol. This method provides comprehensive financial performance metrics for both parties and the latest symbol price.

**Input Parameters**

* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `chainId`: The chain identifier.
* `symbolId`: The identifier of the trading symbol.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `chainId`: The chain identifier.
* `partyB`: The address of partyB.
* `partyA`: The address of partyA.
* `symbolId`: The identifier of the trading symbol.
* `price`: The current price of the specified symbol.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number at the time of the query.
* Additional data included as the `result`.

**Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=uPnlWithSymbolPrice&params[partyB]=$partyB_address&params[partyA]=$partyA_address&params[chainId]=$chainId&params[symbolId]=$symbolId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

#### Example Response

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "symmio",
        "appId": "",
        "method": "uPnlWithSymbolPrice",
        "data": {
            "params": {
                "partyB": "",
                "partyA": "",
                "chainId": "",
                "symbolId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "partyB": "",
                "partyA": "",
                "symbolId": "",
                "price": "",
                "symmio": "",
                "latestBlockNumber": "",
                "uPnlB": "",
                "uPnlA": "",
                "notionalValueSumB": "",
                "notionalValueSumA": "",
                "nonceB": "",
                "nonceA": "",
                "pricesMap": {},
                "maxLeverages": {},
                "markPrices": {
                    "binance": {}
                },
                "pricesB": [],
                "pricesA": [],
                "quoteIdsB": [],
                "quoteIdsA": []
                }
            }
        }
```

### `price`

The `price` method within the Muon API fetches the current prices of specified quotes. This method provides up-to-date pricing information for a list of trading symbols.

**Input Parameters**

* `quoteIds`: A JSON-formatted list of quote identifiers.
* `chainId`: The chain identifier.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number to fetch data for.

**Returned Data Structure**

* `chainId`: The chain identifier.
* `quoteIds`: The list of quote identifiers.
* `symmio`: Address of the Symmio diamond.
* `latestBlockNumber`: The latest block number at the time of the query.
* Additional price data as included in the `result` from `fetchPrices`.

**Example API Query**

```bash
https://muon-oracle4.rasa.capital/v1/?app=symmio&method=price&params[quoteIds]=$quoteIds&params[chainId]=$chainId&params[symmio]=$symmio&params[latestBlockNumber]=$latestBlockNumber
```

#### Example Response:

```json
{
    "success": true,
    "result": {
        "reqId": "",
        "app": "symmio",
        "appId": "",
        "method": "price",
        "data": {
            "params": {
                "quoteIds": "[\"1000\",\"1001\",\"1002\"]",
                "chainId": "",
                "symmio": "",
                "latestBlockNumber": ""
            },
            "timestamp": int,
            "uid": "",
            "result": {
                "chainId": "",
                "quoteIds": [
                    "",
                    "",
                    ""
                ],
                "symmio": "",
                "latestBlockNumber": "",
                "symbols": [
                    "",
                    "",
                    ""
                ],
                "prices": [
                    "",
                    "",
                    ""
                ],
                "pricesMap": {
                }
            }
        }
    }
}
```
