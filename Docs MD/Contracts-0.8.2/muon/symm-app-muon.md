# SYMM App (Muon)

## Introduction

The Muon app is designed to provide signed data for SYMM 3rd Party frontends (with Cloverfield used as an example in this text), a platform that enables users to trade assets and perpetual contracts (perps) on the Ethereum blockchain without the need for Know Your Customer (KYC) procedures. By utilizing their Ethereum wallet, users can participate in trading activities on Cloverfield.

Muon acts as an intermediary service, facilitating the generation and signing of data that is required for trading. This documentation will provide an overview of the app codebase, explaining its key components and functionalities.

## Overview of the Muon App

The Muon app is designed to provide various calculations and information related to unrealized profit and loss (uPnl) and price data. It offers separate uPnl calculations for individual parties, namely "partyA" and "partyB," as well as combined uPnl calculations for both parties together. Additionally, the app allows fetching the prices of given quotes. The following sections provide a detailed explanation of each method provided by the Muon app, showcasing how it calculates uPnl for partyA and partyB separately and together, as well as how it retrieves price information for specified quotes.

## Methods Overview

The Muon app has a set of methods of providing data for contracts:

### `uPnl_A`

The `uPnl_A` method in the Muon app is designed to verify and validate the unrealized profit and loss (uPnl) for partyA based on a set tolerance level. This method ensures that the calculated uPnl does not deviate significantly from an expected value, safeguarding against discrepancies and ensuring accuracy in financial reporting and analysis.

**Returned Data Structure:**

* **`ssymmio`**: Address of the Symmio diamond.
* **`partyA`**: The address of partyA.
* **`nonce`**: Unique nonce for the transaction to prevent replay attacks.
* **`uPnl`**: The verified uPnl value for partyA as obtained from the request data.
* **`timestamp`**: The blockchain timestamp at which the request data was generated.
* **`chainId`**: The chain identifier.

### `partyA_overview`

The `partyA_overview` method in the Muon app serves to validate and provide a comprehensive financial overview for partyA. It checks unrealized profit and loss (uPnl) and loss against predefined tolerance levels, ensuring the reliability of trading data for partyA.

**Returned Data Structure:**

* **`liquidationId`**: Identifier for any liquidation processes that might be relevant to the account.
* **`symmio`**: Address of the Symmio diamond.
* **`partyA`**: The address of partyA.
* **`nonce`**: Unique nonce for the transaction to prevent replay attacks.
* **`uPnl and loss`**: Verified financial metrics showing unrealized profit and losses.
* **`symbolIds`**: Array of symbol IDs related to partyA's positions.
* **`symbolIdsPrices`**: Prices corresponding to the symbol IDs involved.
* **`timestamp and chainId`**: Blockchain-specific metadata ensuring the data's relevance to the correct network and point in time.

### `verify`

The `verify` method in the Muon app is designed to confirm the authenticity and accuracy of liquidation-related data for a particular party (partyA) on the platform. This method plays a critical role in the validation process by packaging and preparing data for further cryptographic checks using Muon's network, specifically targeting the verification of liquidation signatures.

**Returns**:

* **`liquidationId`**: The id of the liquidation.
* **`symmio`**: Address of the Symmio diamond.
* **`verifyLiquidationSig`**: A string indicating the specific verification function to be used.
* **`partyA`**: Ethereum address of the subject party, the primary focus of the verification.
* **`nonce`**: Unique nonce for the transaction to prevent replay attacks.
* **`uPnl and loss`**: Unrealized profits and losses.
* **`symbolIds and prices`**: Lists of trading symbols and their corresponding prices.
* **`timestamp` & `chainId`**: Metadata linking data to the correct blockchain network and timestamp.

### `uPnl_A_withSymbolPrice`

The `uPnl_A_withSymbolPrice` method in the Muon app is designed to calculate the unrealized profit and loss (uPnl) for partyA, associated with a specific trading symbol, and validate the correctness of the symbol's price within predefined tolerance levels. This function is critical for ensuring accurate and timely financial reporting for trading activities.

**Returns:**

* **`symmio`**: Address of the Symmio diamond.
* **`partyA`**: Address of the trader whose uPnl is being computed.
* **`nonce`**: Unique nonce for the transaction to prevent replay attacks.
* **`uPnl`**: Calculated unrealized profit or loss for the specified positions of partyA.
* **`symbolId`**: Identifier for the trading symbol whose price is verified.
* **`price`**: Verified price of the symbol.
* **`timestamp` & `chainId`**: Metadata linking data to the correct blockchain network and timestamp.

### `uPnl_B`

The `uPnl_B` method within the Muon app calculates the unrealized profit and loss (uPnl) specifically for partyB in relation to partyA.&#x20;

**Response Structure**:

* **`symmio`**: Address of the Symmio diamond.
* **`partyB`**: Address of the solver (partyB) whose uPnl is evaluated.
* **`partyA`**: Address of the associated party (partyA) involved in the transaction.
* **`nonce`**: Unique nonce for the transaction to prevent replay attacks.
* **`uPnl`**: The computed unrealized profit or loss for partyB's positions, providing a snapshot of financial performance.
* **`timestamp` & `chainId`**: Metadata linking data to the correct blockchain network and timestamp.

### `uPnl`

The `uPnl` method within the Muon app calculates and verifies the unrealized profit and loss (uPnl) for both partyA and partyB. This method facilitates a comprehensive assessment of financial performance for both parties involved in the trading activities on decentralized platforms.

**Response Structure**:

* **`symmio`**: Address of the Symmio diamond.
* **`partyB` and `partyA`**: Addresses of the involved parties.
* **`nonceB` and `nonceA`**: Nonce values for both parties .
* **`uPnlB` and `uPnlA`**: The computed unrealized profit or loss for both parties.
* **`timestamp` & `chainId`**: Metadata linking data to the correct blockchain network and timestamp.

### `uPnlWithSymbolPrice`

The `uPnlWithSymbolPrice` method in the Muon app calculates and verifies the unrealized profit and loss (uPnl) for both partyA and partyB, incorporating the price of a specific symbol to enrich the financial data context.

**Response Components**:

* **`symmio`**: Address of the Symmio diamond.
* **`partyB` and `partyA`**: Theaddresses of the involved parties.
* **`nonceB` and `nonceA`**: The nonces of partyA and partyB
* **`uPnlB` and `uPnlA`**: The computed unrealized profit or loss for both parties.
* **`symbolId` and `price`**: The identifier for the specific trading symbol and its corresponding price.
* **`timestamp` & `chainId`**: Metadata linking data to the correct blockchain network and timestamp.

### `price`

The `price` method in the Muon app retrieves and validates the prices of multiple quotes, ensuring they align within specified tolerances. This method is essential for confirming real-time asset pricing accuracy on decentralized trading platforms.

\
**Response Components**:&#x20;

* **`symmio`**: Address of the Symmio diamond.
* **`quoteIds`**: An array of quote ids.
* **`prices`**: The validated list of prices corresponding to the quote IDs.
* **`timestamp`**: The timestamp marking the retrieval of the price data.
* **`chainId`**: The chain identifier.

## Functions Breakdown

### uPnlPartyA

The `uPnlPartyA` function in the Muon app calculates the unrealized profit and loss (uPnl) for partyA. Here's a breakdown of its implementation:

```javascript
uPnlPartyA: async function (partyA, chainId, v3Contract) {
    // Fetches the open positions and quote IDs for partyA
    const { openPositions, quoteIds, symbolIds } = await this.fetchOpenPositions({ partyA }, 'A', chainId, v3Contract);

    // Retrieves the nonce of partyA
    const nonce = await ethCall(v3Contract, 'nonceOfPartyA', [partyA], ABI, chainId);

    // If there are no open positions, return the result with zero uPnl, notional value sum,
    // nonce, quote IDs, open positions, prices map and mark prices (retrieved using the getPrices function)
    if (openPositions.length == 0) {
        const { pricesMap, markPrices } = await this.getPrices([]);
        return {
            uPnl: ZERO.toString(),
            loss: ZERO.toString(),
            notionalValueSum: ZERO.toString(),
            nonce,
            quoteIds,
            symbolIds: [],
            prices: [],
            openPositions,
            pricesMap,
            markPrices
        };
    }

    // Fetches the prices, prices map and mark prices for the quote IDs
    const { symbols, prices, pricesMap, markPrices } = await this.fetchPrices(quoteIds, chainId, v3Contract);

    // Calculates the uPnl and notional value sum using the open positions and prices
    const { uPnl, loss, notionalValueSum } = await this.calculateUpnl(openPositions, prices);

    // Returns the result with the calculated uPnl, notional value sum, nonce, prices map,
    // prices, quote IDs, open positions and mark prices
    return {
        uPnl: uPnl.toString(),
        loss: loss.toString(),
        notionalValueSum: notionalValueSum.toString(),
        nonce,
        pricesMap,
        symbolIds,
        symbols,
        prices,
        quoteIds,
        openPositions,
        markPrices
    };
},
```

This function fetches the open positions and quote IDs for partyA, retrieves the nonce of partyA, and calculates the uPnl and notional value sum using the open positions and corresponding prices. If there are no open positions, it returns a result with zero uPnl and notional value sum, along with other relevant information such as nonce, quote IDs, open positions, prices map and markPrices.

### uPnlPartyB

The `uPnlPartyB` function in the Muon app calculates the unrealized profit and loss (uPnl) for partyB, given the associated partyA. Here's a breakdown of its implementation:

```javascript
uPnlPartyB: async function (partyB, partyA, chainId, v3Contract) {
    // Fetches the open positions and quote IDs for partyB with the associated partyA
    const { openPositions, quoteIds } = await this.fetchOpenPositions({ partyB, partyA }, 'B', chainId, v3Contract);
    
    // Retrieves the nonce of partyB for the given partyA
    const nonce = await ethCall(v3Contract, 'nonceOfPartyB', [partyB, partyA], ABI, chainId);
    
    // If there are no open positions, return the result with zero uPnl, notional value sum,
    // nonce, and quote IDs
    if (openPositions.length == 0) {
        return {
            uPnl: ZERO.toString(),
            notionalValueSum: ZERO.toString(),
            nonce,
            quoteIds
        };
    }
    
    // Fetches the prices for the quote IDs and prices map for all symbols
    const { prices, pricesMap } = await this.fetchPrices(quoteIds, chainId, v3Contract);
    
    // Calculates the uPnl and notional value sum using the open positions and prices
    const { uPnl, notionalValueSum } = await this.calculateUpnl(openPositions, prices);

    // Returns the result with the calculated uPnl (multiplied by -1 to represent partyB's perspective),
    // notional value sum, nonce, prices map, prices, and quote IDs
    return {
        uPnl: minusOne.mul(uPnl).toString(),
        notionalValueSum: notionalValueSum.toString(),
        nonce,
        pricesMap,
        prices,
        quoteIds
    };
}
```

This function fetches the open positions and quote IDs for partyB with the associated partyA, retrieves the nonce of partyB for the given partyA, and calculates the uPnl and notional value sum using the open positions and corresponding prices. If there are no open positions, it returns a result with zero uPnl and notional value sum, along with other relevant information such as nonce and quote IDs. The calculated uPnl is multiplied by -1 to represent partyB's perspective.

### uPnlPartyB\_FetchedData

The `uPnlPartyB_FetchedData` function in the Muon app calculates the unrealized profit and loss (uPnl) for partyB using fetched data. Here's a breakdown of its implementation:

```javascript
uPnlPartyB_FetchedData: async function (partyB, partyA, chainId, pricesMap, mixedOpenPositions, v3Contract) {
    // Filters the mixed open positions to only include positions associated with partyB
    const { openPositions, quoteIds } = this.filterPositions(partyB, mixedOpenPositions);

    let uPnl, notionalValueSum, prices;
    
    if (openPositions.length > 0) {
        // Retrieves the symbols associated with the quote IDs
        const symbols = await this.getSymbols(quoteIds, chainId, v3Contract);
        
        // Creates a prices list using the symbols and prices map
        prices = this.createPricesList(symbols, pricesMap);
        
        // Calculates the uPnl and notional value sum using the open positions and prices
        const result = await this.calculateUpnl(openPositions, prices);
        uPnl = result.uPnl;
        notionalValueSum = result.notionalValueSum;
    } else {
        // If there are no open positions, set uPnl and notional value sum to zero
        uPnl = notionalValueSum = new BN(0);
        prices = [];
    }

    // Retrieves the nonce of partyB for the given partyA
    const nonce = await ethCall(v3Contract, 'nonceOfPartyB', [partyB, partyA], ABI, chainId);

    // Returns the result with the calculated uPnl, notional value sum, nonce, prices, and quote IDs
    return {
        uPnl,
        notionalValueSum,
        nonce,
        prices,
        quoteIds
    };
}
```

This function filters the mixed open positions to include only the positions associated with partyB. It then checks if there are any open positions remaining. If there are open positions, it retrieves the symbols associated with the quote IDs, creates a prices list using the symbols and prices map, and calculates the uPnl and notional value sum using the open positions and prices. If there are no open positions, it sets the uPnl and notional value sum to zero. Finally, it retrieves the nonce of partyB for the given partyA and returns the result with the calculated uPnl, notional value sum, nonce, prices, and quote IDs.

### uPnlParties

The `uPnlParties` function in the Muon app calculates the unrealized profit and loss (uPnl) for both partyA and partyB and returns the results. Here's a breakdown of its implementation:

```javascript
uPnlParties: async function (partyB, partyA, chainId, v3Contract) {
    // Checks if partyB and partyA are identical, and throws an error if they are
    if (partyB == partyA) {
        throw { message: 'Identical Parties Error' };
    }

    // Calculates the uPnl, nonce, notional value sum, prices map, prices, and quote IDs for partyA
    const { uPnl: uPnlA, nonce: nonceA, notionalValueSum: notionalValueSumA, pricesMap, prices: pricesA, quoteIds: quoteIdsA, openPositions } = await this.uPnlPartyA(partyA, chainId, v3Contract);

    // Calculates the uPnl, nonce, notional value sum, prices, and quote IDs for partyB using fetched data
    const { uPnl: uPnlB, nonce: nonceB, notionalValueSum: notionalValueSumB, prices: pricesB, quoteIds: quoteIdsB } = await this.uPnlPartyB_FetchedData(partyB, partyA, chainId, pricesMap, openPositions, v3Contract);

    // Returns the results with adjusted uPnl for partyB, uPnl for partyA, notional value sum for partyB and partyA,
    // nonces for partyB and partyA, prices map, prices for partyB and partyA, and quote IDs for partyB and partyA
    return {
        uPnlB: minusOne.mul(uPnlB).toString(),
        uPnlA,
        notionalValueSumB: notionalValueSumB.toString(),
        notionalValueSumA,
        nonceB,
        nonceA,
        pricesMap,
        pricesB,
        pricesA,
        quoteIdsB,
        quoteIdsA
    };
}
```

This function first checks if partyB and partyA are identical and throws an error if they are. Then, it calls the uPnlPartyA function to calculate the uPnl, nonce, notional value sum, prices map, prices, and quote IDs for partyA. Next, it calls the uPnlPartyB\_FetchedData function to calculate the uPnl, nonce, notional value sum, prices, and quote IDs for partyB using fetched data. Finally, it returns the results with the adjusted uPnl for partyB (multiplied by -1), uPnl for partyA, notional value sum for partyB and partyA, nonces for partyB and partyA, prices map, prices for partyB and partyA, and quote IDs for partyB and partyA.

### calculateUpnl

The `calculateUpnl` function in the Muon app is responsible for calculating the unrealized profit and loss (uPnl) and the notional value sum for a given set of open positions and their corresponding prices. Here's an explanation of how this function works:

```javascript
calculateUpnl: async function (openPositions, prices) {
    let uPnl = new BN(0); // Initializes uPnl to zero
    let loss = new BN(0); // Initializes loss to zero
    let notionalValueSum = new BN(0); // Initializes notionalValueSum to zero

    // Iterates through each open position
    for (let [i, position] of openPositions.entries()) {
        const openedPrice = new BN(position.openedPrice); // Retrieves the opened price of the position
        const priceDiff = new BN(prices[i]).sub(openedPrice); // Calculates the price difference between the current price and the opened price
        const amount = new BN(position.quantity).sub(new BN(position.closedAmount)); // Calculates the remaining amount of the position

        // Calculates the uPnl for the current position based on the position type (long or short)
        const longPositionUpnl = amount.mul(priceDiff);
        const positionUpnl = position.positionType == '0' ? longPositionUpnl : minusOne.mul(longPositionUpnl);

        // Adds the position's uPnl to the total uPnl after scaling it
        uPnl = uPnl.add(positionUpnl.div(scale));
        // Add the position's uPnl to the total loss if it is negative
        if (positionUpnl.isNeg()) loss = loss.add(positionUpnl.div(scale));

        // Calculates the notional value of the position and adds it to the total notional value sum
        const positionNotionalValue = amount.mul(openedPrice).div(scale);
        notionalValueSum = notionalValueSum.add(positionNotionalValue);
    }

    // Returns the calculated uPnl and notional value sum
    return { uPnl, loss, notionalValueSum };
},
```

This function iterates through each open position and calculates the uPnl for each position by multiplying the remaining amount of the position by the price difference. The uPnl is adjusted based on the position type (long or short) and scaled before being added to the total uPnl. Additionally, the notional value of each position is calculated by multiplying the remaining amount by the opened price and scaled before being added to the total notional value sum. Finally, the function returns the calculated uPnl and notional value sum as an object.

### fetchPrices

The `fetchPrices` method obtains the latest prices for a set of quoteIds by fetching the symbols and corresponding prices. It returns the prices as an array and creates a map for convenient access within the Muon app. Here's a breakdown of its implementation:

```javascript
fetchPrices: async function (quoteIds, chainId, v3Contract) {
    // Retrieve symbols corresponding to the given quoteIds
    const symbols = await this.getSymbols(quoteIds, chainId, v3Contract);

    // Fetch the latest prices and create a prices map
    const { pricesMap, markPrices } = await this.getPrices(symbols);

    // Create an array of prices by matching symbols with prices in the map
    const prices = this.createPricesList(symbols, pricesMap);

    // Return an object containing the prices array and prices map
    return { symbols, prices, pricesMap, markPrices };
},
```

This function is responsible for retrieving prices corresponding to a given set of quoteIds. It utilizes other helper methods to fetch symbols, obtain the latest prices, and create a map of prices. By combining these components, the method returns an object containing an array of prices and a map that associates symbols with their respective prices. This enables the Muon app to efficiently access and utilize the required price data for contract-related operations.

### getPrices

```javascript
getPrices: async function (symbols) {
    // Create promises for retrieving prices from different exchanges
    const promises = [
        this.getBinancePrices(),  // Promise for Binance prices
        this.getKucoinPrices(),   // Promise for Kucoin prices
        this.getMexcPrices(),     // Promise for Mexc prices
    ]

    // Wait for all promises to resolve and get the results
    const result = await Promise.all(promises)

    // Store the prices in the markPrices object with exchange names as keys
    const markPrices = {
        'binance': result[0],   // Binance prices
        'kucoin': result[1],    // Kucoin prices
        'mexc': result[2],      // Mexc prices
    }

    // Check the retrieved prices for any corruption
    if (!this.checkPrices(symbols, markPrices)) {
        throw { message: `Corrupted Price` }
    }

    // Return the pricesMap and markPrices object
    return { pricesMap: markPrices['binance'], markPrices }
}
```

In this implementation, the `getPrices` function uses three separate functions (`getBinancePrices`, `getKucoinPrices`, and `getMexcPrices`) to fetch prices from different exchanges asynchronously. The results are then stored in the markPrices object, which includes the prices for each exchange. The function also checks the retrieved prices using the checkPrices function to ensure their validity. Finally, the function returns an object containing the pricesMap and markPrices for further usage.

### getBinancePrices

The `getBinancePrices` function retrieves the latest price data for various symbols from binance API. It makes an HTTP request to the API endpoint and processes the response to extract the symbol-price mappings. The extracted data is stored in an object, which is then returned by the function. Here's a breakdown of its implementation:

```javascript
getBinancePrices: async function () {
    // Define the Binance API URL
    const binanceUrl = 'https://fapi.binance.com/fapi/v1/premiumIndex';

    // Make an HTTP GET request to the Binance API using Axios
    const { data } = await axios.get(binanceUrl, {
        proxy: false,
        httpsAgent: new HttpsProxyAgent.HttpsProxyAgent(proxy)
    });

    // Create an empty object to store the prices map
    const pricesMap = {};

    // Iterate over the data received from the API
    data.forEach((el) => {
        // Convert the mark price to a string and store it in the prices map
        pricesMap[el.symbol] = scaleUp(el.markPrice).toString();
    });

    // Return the populated prices map
    return pricesMap;
},
```

This function implements a logic to fetch the latest price data for different symbols from an external API. It uses an HTTP request to the API endpoint and processes the response to extract the symbol-price mappings. The extracted data is stored in an object, which is returned by the function. This allows the application to access up-to-date price information for various symbols, enabling accurate calculations and analysis in the app.

### checkPrices

```javascript
checkPrices: function (symbols, markPrices) {
    // Get the expected prices from the Binance markPrices
    const expectedPrices = markPrices['binance']

    // Iterate over each symbol in the given symbols array
    for (let symbol of symbols) {
        // Get the expected price for the symbol
        const expectedPrice = expectedPrices[symbol]

        // Throw an error if the expected price is undefined
        if (expectedPrice == undefined) {
            throw { message: 'Undefined Binance Price' }
        }

        // Iterate over each source (Kucoin and Mexc) to compare their prices
        for (let source of ['kucoin', 'mexc']) {
            // Get the pricesMap for the source exchange
            const pricesMap = markPrices[source]

            // Get the price for the symbol from the pricesMap
            let price = pricesMap[symbol]

            // Throw an error if the price is undefined (unsupported symbol)
            if (price == undefined) {
                throw { message: 'Unsupported Symbol' }
            }

            // Check if the price tolerance is within the acceptable range
            if (!this.isPriceToleranceOk(price, expectedPrice, PRICE_TOLERANCE).isOk) {
                return false
            }
        }
    }

    // All prices are within tolerance, return true
    return true
}
```

The `checkPrices` function is responsible for verifying the validity of the fetched prices. It compares the prices of different sources (Kucoin and Mexc) with the expected prices from Binance. The function iterates over each symbol and checks if the expected price exists for the symbol in the Binance prices. If not, an error is thrown. Then, it retrieves the prices for the symbol from each source and checks if they fall within the acceptable price tolerance range using the `isPriceToleranceOk` function. If any price is outside the tolerance range, the function returns `false`. Otherwise, if all prices pass the tolerance check, it returns `true`, indicating that the prices are valid.

### fetchPartyBsAllocateds

This function retrieves the allocated balances for multiple `partyB` accounts with respect to a single `partyA`. It queries the Symmio smart contract to obtain the amount of funds each `partyB` has allocated towards trading with `partyA`.

```javascript
    fetchPartyBsAllocateds: async function (chainId, symmio, partyA, partyBs, blockNumber) {
        const allocateds = await ethCall(symmio, 'allocatedBalanceOfPartyBs', [partyA, partyBs], ABI, chainId, blockNumber)
        return allocateds
    },
```

Returns an array of allocated balance amounts. Each element in the array corresponds to the allocated balance of a `partyB` as indexed in the `partyBs` array.&#x20;

### filterPositions

This function filters a mixed array of open trading positions to extract those that involve a specific `partyB`. It is used to segregate positions by the counterparty involved, making it easier to manage or analyze the trading activities related to a particular `partyB`.

```javascript
    filterPositions: function (partyB, mixedOpenPositions) {
        let quoteIds = []
        let openPositions = []
        mixedOpenPositions.forEach((position) => {
            if (position.partyB == partyB) {
                openPositions.push(position)
                quoteIds.push(String(position.id))
            }
        })
        return { openPositions, quoteIds }
    },
```

**Returns:**: An object containing two properties:

* **openPositions** (`Array`): An array of all positions that match the specified `partyB`.
* **quoteIds** (`Array`): An array of quoteIds corresponding to the positions.

### isPriceToleranceOk

This function evaluates whether the deviation of a given price from an expected price falls within an acceptable tolerance range. It is typically used to ensure that price fluctuations remain within predefined limits, which can be crucial for trading strategies that require price accuracy and stability.

```javascript
isPriceToleranceOk: function (price, expectedPrice, priceTolerance) {
        let priceDiff = new BN(price).sub(new BN(expectedPrice)).abs()
        const priceDiffPercentage = new BN(priceDiff).mul(scale).div(new BN(expectedPrice))
        return {
            isOk: !priceDiffPercentage.gt(new BN(priceTolerance)),
            priceDiffPercentage: parseFloat(priceDiffPercentage.mul(new BN(10000)).div(scale).toString()) / 100
        }
    },
```

**Returns:**

* **Object**: An object containing:
  * **isOk** (`Boolean`): A boolean indicating whether the actual price is within the specified tolerance of the expected price.
  * **priceDiffPercentage** (`Float`): The calculated price difference as a percentage, scaled to two decimal places for readability.

### isUpnlToleranceOk

This function determines whether the difference between an actual unrealized profit and loss (uPnl) and an expected uPnl is within a specified tolerance, relative to the notional value of the position. It is crucial in financial analysis to ensure the uPnl remains within acceptable risk parameters.

```javascript
    isUpnlToleranceOk: function (uPnl, expectedUpnl, notionalValueSum, uPnlTolerance) {
        if (new BN(notionalValueSum).eq(ZERO))
            return { isOk: new BN(expectedUpnl).eq(ZERO) }

        let uPnlDiff = new BN(uPnl).sub(new BN(expectedUpnl)).abs()
        const uPnlDiffInNotionalValue = uPnlDiff.mul(scale).div(new BN(notionalValueSum))
        return {
            isOk: !uPnlDiffInNotionalValue.gt(new BN(uPnlTolerance)),
            uPnlDiffInNotionalValue: uPnlDiffInNotionalValue.mul(new BN(100)).div(scale)
        }
    },

```

**Returns**: An object containing:

* **isOk** (`Boolean`): A boolean indicating whether the uPnl is within the specified tolerance relative to the notional value.
* **uPnlDiffInNotionalValue** (`BigNumber`): The uPnl difference expressed as a percentage of the notional value, adjusted for readability (multiplied by 100 for percentage format).

### getSymbols

This function retrieves the symbols associated with given quote IDs by querying the symmio contract. It's particularly useful in trading and financial platforms where understanding the market symbols related to specific trading quotes is necessary for data display or further processing.

```javascript
getSymbols: async function (quoteIds, chainId, symmio, blockNumber) {
        const symbols = await ethCall(symmio, 'symbolNameByQuoteId', [quoteIds], ABI, chainId, blockNumber)
        if (symbols.includes('')) throw { message: 'Invalid quoteId' }
        return symbols
    },
```

* **Returns:**
* **Array** (`String[]`): Returns an array of symbol names corresponding to the quote IDs provided. Each symbol in the array aligns with the quote ID at the same index in the input array.

### fetchOpenPositions

This function retrieves all open trading positions for a specified party (`partyA` or `partyB`) from the Symmio platform, spanning potentially multiple pages of data. It compiles an exhaustive list of positions, including corresponding quote IDs, symbol IDs, and involved party B addresses.

```javascript
    fetchOpenPositions: async function (parties, side, chainId, symmio, blockNumber) {
        const positionsCount = new BN(await this.getPositionsCount(parties, side, chainId, symmio, blockNumber))
        if (positionsCount.eq(new BN(0))) return { openPositions: [], quoteIds: [] }

        const size = 50
        const getsCount = parseInt(positionsCount.div(new BN(size))) + 1

        const openPositions = []
        for (let i = 0; i < getsCount; i++) {
            const start = i * size
            openPositions.push(...await this.getOpenPositions(parties, side, start, size, chainId, symmio))
        }

        let quoteIds = []
        let symbolIds = new Set()
        let partyBs = new Set()
        openPositions.forEach((position) => {
            quoteIds.push(String(position.id))
            symbolIds.add(String(position.symbolId))
            partyBs.add(position.partyB)
        })

        symbolIds = Array.from(symbolIds)
        partyBs = Array.from(partyBs)

        return {
            openPositions,
            quoteIds,
            symbolIds,
            partyBs,
        }
    },

```

This function returns an object containing arrays of open positions, quote IDs, symbol IDs, and party B addresses:

```javascript
{
    openPositions: Array,
    quoteIds: Array,
    symbolIds: Array,
    partyBs: Array
}
```

### getOpenPositions

This function retrieves a list of open trading positions for a specified party (`partyA` or `partyB`) from the Symmio platform. It dynamically selects the appropriate Symmio contract method based on whether the positions for `partyA` or `partyB` are requested.

```javascript
   getOpenPositions: async function (parties, side, start, size, chainId, symmio, blockNumber) {
        if (side == 'A') return await ethCall(symmio, 'getPartyAOpenPositions', [parties.partyA, start, size], ABI, chainId, blockNumber)
        else if (side == 'B') return await ethCall(symmio, 'getPartyBOpenPositions', [parties.partyB, parties.partyA, start, size], ABI, chainId, blockNumber)
    },
```

This function supports pagination through the `start` and `size` parameters and accommodates historical data queries with the `blockNumber` parameter.

### getPositionsCount

This function retrieves the count of open positions for a specified party (`partyA` or `partyB`) on the Symmio platform. It determines which function to call based on the side (`A` or `B`) indicated, making it a versatile tool for querying position counts in a decentralized trading environment.

```javascript
    getPositionsCount: async function (parties, side, chainId, symmio, blockNumber) {
        if (side == 'A') return await ethCall(symmio, 'partyAPositionsCount', [parties.partyA], ABI, chainId, blockNumber)
        else if (side == 'B') return await ethCall(symmio, 'partyBPositionsCount', [parties.partyB, parties.partyA], ABI, chainId, blockNumber)
    },

```

**Returns**: `Number`, the count of open positions for the specified party at the given block number.

### createPricesList

This function creates a list of prices for specified cryptocurrency symbols based on a mapping of symbol-to-price

```javascript
    createPricesList: function (symbols, pricesMap) {
        const prices = []
        symbols.forEach((symbol) => prices.push(pricesMap[symbol].toString()))
        return prices
    },
```

**Returns**: `prices` (`Array`), an array containing the price values as strings for the requested symbols.

### getMexcPrices

The `getMexcPrices` function in the Muon app retrieves current cryptocurrency prices from the MEXC exchange. Here's a detailed breakdown of its implementation:

```javascript
    getMexcPrices: async function () {
        const mexcUrl = priceUrl + '/mexc'
        const { data } = await axios.get(mexcUrl)
        const pricesMap = {}
        data.forEach((el) => {
            const symbol = el.symbol.replace('_', '')
            pricesMap[symbol] = scaleUp(Number(el.price).toFixed(10)).toString()
        })

        for (let symbol of mexcThousandPairs)
            pricesMap['1000' + symbol] = new BN(pricesMap[symbol]).mul(new BN(1000)).toString()
        pricesMap['LUNA2USDT'] = pricesMap['LUNANEWUSDT']
        pricesMap['FILUSDT'] = pricesMap['FILECOINUSDT']
        pricesMap['BNXUSDT'] = pricesMap['BNXNEWUSDT']
        return pricesMap
    },

```

This function fetches the latest trading prices for various cryptocurrency pairs from the MEXC exchange. It processes the received data to ensure that prices are formatted correctly and returned in a map of symbol-to-price associations tailored for further processing and display.

**Returns**: `pricesMap`, an object containing key-value pairs where keys are cryptocurrency symbols (e.g., `BTCUSDT`, `ETHUSDT`) and values are their corresponding prices, adjusted and scaled appropriately.

### getKucoinPrices

The `getKucoinPrices` function in the Muon app fetches current cryptocurrency prices from the Kucoin exchange. Here's a detailed breakdown of its implementation:

```javascript
    getKucoinPrices: async function () {
        const kucoinUrl = priceUrl + '/kucoin'
        const { data } = await axios.get(kucoinUrl, {
            headers: { "Accept-Encoding": "gzip,deflate,compress" }
        })
        const pricesMap = {}
        data.forEach((el) => {
            try {
                const symbol = this.formatKucoinSymbol(el)
                pricesMap[symbol] = scaleUp(Number(el.price).toFixed(10)).toString()
            }
            catch (e) { }
        })
        pricesMap['BTCUSDT'] = pricesMap['XBTUSDT']
        for (let symbol of kucoinThousandPairs) {
            if (pricesMap[symbol] == undefined) continue
            pricesMap['1000' + symbol] = new BN(pricesMap[symbol]).mul(new BN(1000)).toString()
        }
        pricesMap['LUNA2USDT'] = pricesMap['LUNAUSDT']
        return pricesMap
    },

```

**Returns**: An object (`pricesMap`) containing key-value pairs where keys are the cryptocurrency symbols (e.g., `BTCUSDT`, `ETHUSDT`) and values are their corresponding prices, adjusted and scaled as necessary.

### formatKucoinSymbol

This function formats a cryptocurrency symbol from Kucoin to standardize it for use within the application

```javascript
    formatKucoinSymbol: function (kucoinPair) {
        let symbol = kucoinPair.symbol.slice(0, -1)
        return symbol
    },
```

**Returns**: `symbol` (`String`), a string representing the formatted cryptocurrency symbol without the last character.

## Summary

To summarize the Muon app's capabilities and purpose, it offers a range of methods tailored specifically for contract-related scenarios. These methods play a vital role in providing the necessary signed data for SYMM 3rd party frontends (e.g. Cloverfield) smart contracts. By enabling functions such as fetching open positions, calculating uPnl, and retrieving external price data, the app facilitates seamless interaction with contracts, empowering users and developers to effectively manage and monitor their investments. With a strong emphasis on contract use cases, the Muon app serves as a valuable tool for securely integrating blockchain technology into various applications.\
