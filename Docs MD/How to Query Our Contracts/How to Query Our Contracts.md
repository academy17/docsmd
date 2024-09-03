# How to Query our Contracts

## The SYMM Diamond

SYMMIO's core contracts are implemented with **EIP-2535**, a solution for building modular and upgradeable contracts.

#### **EIP-2535: Diamond Standard Overview** <a href="#eip-2535-diamond-standard-overview" id="eip-2535-diamond-standard-overview"></a>

EIP-2535 introduces the Diamond Standard in smart contract development. This standard centralizes around the concept of a "Diamond" contract, which is constructed from multiple "facets," each responsible for different functionalities. This modular design enhances maintainability by allowing acets to be upgraded or modified independently without affecting the entire contract. Key to this architecture are function selectors, which serve as unique identifiers for each function within the facets. These selectors efficiently direct function calls to the correct facet, facilitating organized and conflict-free code execution. For further reading and technical details, refer to the [official EIP-2535 documentation](https://eips.ethereum.org/EIPS/eip-2535).

When a facet, is invoked via a `delegatecall` from the Diamond, it operates directly on the Diamond’s storage. The facet uses the slot address computed by the library to access or modify the data in the specific storage slot. This operation is transparent to the Diamond itself.

### Finding Facet Contracts

In the SYMM Diamond, the `facets()` function of the `DiamondLoupeFacet` plays a crucial role in inspecting the contract's structure and behaviour. When this function is called on the Diamond, it returns an array of `Facet` structs, each containing the address of a facet along with the specific function selectors associated with that facet.&#x20;

To examine the facets and call functions on them, we'll use a tool called [Louper](https://louper.dev). This platform displays all facets and functions related to a Diamond contract and allows us to query data directly from them. Since functions cannot be called on through the Diamond contract directly through a blockchain explorer, Louper provides an efficient and user-friendly way to interact with the Diamond.

### Using Louper to Read Data

In this example, we'll use Louper to find all the contract symbols available to trade on Thena.

First, paste the address into the tool and select the appropriate chain (BNB):



<figure><img src=".gitbook/assets/louper1.png" alt=""><figcaption></figcaption></figure>

Then, click the **Read** tab, and select the View Facet. This shows all the view functions available to query on the Diamond.



<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Now you can query the symbols available to trade on the Diamond using `getSymbols()` with `start` and `size` as parameters:



<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

For more information on the View Facet's functions, please refer to the [ViewFacet ](https://app.gitbook.com/o/-McFr-NnfqafWnoj\_Y5M/s/YQhBmTCs9MwhhuPQrV5v/\~/changes/474/building-on-symm-io/technical-documentation/contracts-documentation-0.8.2/facets/view-facet)documentation (0.8.2).&#x20;

## Using Subgraphs to Query Data

The Graph is a decentralized protocol for indexing and querying blockchain data. You can also query data on the diamond using a subgraph URL provided by SYMM.  In this example, we'll query the symbol data using the Apollo Client.

**Prerequisites**

* Basic understanding of GraphQL
* A tool for making HTTP requests, such as Postman, cURL, or a GraphQL client like Apollo Client

**Endpoint URL**

First identify a suitable endpoint:

```
https://api.studio.thegraph.com/query/62454/analytics_bnb_8_2/version/latest
```

This endpoint is the analytics endpoint for the SYMMIO Diamond on BNB.

**Example Query**

To fetch data, send a POST request to the endpoint with the GraphQL query:

```graphql
query MyQuery {
  symbols(first: 3) {
    blockNumber
    id
    name
    timestamp
    tradingFee
    updateTimestamp
  }
}
```

**Using cURL**

```sh
shCopy codecurl -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"query MyQuery { symbols(first: 3) { blockNumber id name timestamp tradingFee updateTimestamp } }"}' \
  https://api.studio.thegraph.com/query/62454/analytics_bnb_8_2/version/latest
```

### **Using Apollo Client in React**

#### Install Apollo Client:

```bash
npm install @apollo/client graphql
```

#### Set up Apollo Client and make the query:

```javascript
import React from 'react';
import { ApolloClient, InMemoryCache, ApolloProvider, useQuery, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://api.studio.thegraph.com/query/62454/analytics_bnb_8_2/version/latest',
  cache: new InMemoryCache()
});

const GET_SYMBOLS = gql`
  query MyQuery {
    symbols(first: 3) {
      blockNumber
      id
      name
      timestamp
      tradingFee
      updateTimestamp
    }
  }
`;

const DataComponent = () => {
  const { loading, error, data } = useQuery(GET_SYMBOLS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return (
    <div>
      <h2>Symbols</h2>
      {data.symbols.map(symbol => (
        <div key={symbol.id}>
          <p>{symbol.name}</p>
          <p>Block Number: {symbol.blockNumber}</p>
          <p>Timestamp: {new Date(symbol.timestamp * 1000).toLocaleString()}</p>
          <p>Trading Fee: {symbol.tradingFee}</p>
          <p>Update Timestamp: {new Date(symbol.updateTimestamp * 1000).toLocaleString()}</p>
        </div>
      ))}
    </div>
  );
};

const App = () => (
  <ApolloProvider client={client}>
    <DataComponent />
  </ApolloProvider>
);

export default App;
```

## Subgraph URLs by Chain

### Arbitrum

#### Main Subgraph

[https://api.studio.thegraph.com/query/62454/main\_arbitrum\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/main\_arbitrum\_8\_2/version/latest)

#### Analytics Subgraph

[https://api.studio.thegraph.com/query/62454/analytics\_arbitrum\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/analytics\_arbitrum\_8\_2/version/latest)

#### Parties Subgraph

[https://api.studio.thegraph.com/query/62454/parties\_arbitrum\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/parties\_arbitrum\_8\_2/version/latest)

#### Funding Rate Subgraph

[https://api.studio.thegraph.com/query/62454/fundingrate\_arbitrum\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/fundingrate\_arbitrum\_8\_2/version/latest)

#### Events Subgraph

[https://api.studio.thegraph.com/query/62454/events\_arbitrum\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/events\_arbitrum\_8\_2/version/latest)

### Mantle

#### Main Subgraph

[https://subgraph-api.mantle.xyz/subgraphs/name/main\_mantle\_8\_2](https://subgraph-api.mantle.xyz/subgraphs/name/main\_mantle\_8\_2)

#### Analytics Subgraph

[https://subgraph-api.mantle.xyz/subgraphs/name/analytics\_mantle\_8\_2/graphql](https://subgraph-api.mantle.xyz/subgraphs/name/analytics\_mantle\_8\_2/graphql)

#### Parties Subgraph

[https://subgraph-api.mantle.xyz/subgraphs/name/parties\_mantle\_8\_2/graphql](https://subgraph-api.mantle.xyz/subgraphs/name/parties\_mantle\_8\_2/graphql)

#### Funding Rate Subgraph

[https://subgraph-api.mantle.xyz/subgraphs/name/parties\_mantle\_8\_2/graphql](https://subgraph-api.mantle.xyz/subgraphs/name/parties\_mantle\_8\_2/graphql)

### Blast

#### Main Subgraph

[https://api.studio.thegraph.com/query/62454/main\_blast\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/main\_blast\_8\_2/version/latest)

#### Analytics Subgraph

[https://api.studio.thegraph.com/query/62454/analytics\_blast\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/analytics\_blast\_8\_2/version/latest)

#### Parties Subgraph

[https://api.studio.thegraph.com/query/62454/parties\_blast\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/parties\_blast\_8\_2/version/latest)

#### Funding Rate Subgraph

[https://api.studio.thegraph.com/query/62454/fundingrate\_blast\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/fundingrate\_blast\_8\_2/version/latest)

#### Events Subgraph

[https://api.studio.thegraph.com/query/62454/events\_blast\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/events\_blast\_8\_2/version/latest)

### BNB

#### Main Subgraph

[https://api.studio.thegraph.com/query/62454/main\_bnb\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/main\_bnb\_8\_2/version/latest)

#### Analytics Subgraph

[https://api.studio.thegraph.com/query/62454/analytics\_bnb\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/analytics\_bnb\_8\_2/version/latest)

#### Parties Subgraph

[https://api.studio.thegraph.com/query/62454/parties\_bnb\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/parties\_bnb\_8\_2/version/latest)

#### Funding Rate Subgraph

[https://api.studio.thegraph.com/query/62454/fundingrate\_bnb\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/fundingrate\_bnb\_8\_2/version/latest)

#### Events Subgraph

[https://api.studio.thegraph.com/query/62454/events\_bnb\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/events\_bnb\_8\_2/version/latest)

### Base

#### Main Subgraph

[https://api.studio.thegraph.com/query/62454/main\_base\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/main\_base\_8\_2/version/latest)

#### Analytics Subgraph

[https://api.studio.thegraph.com/query/62454/analytics\_base\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/analytics\_base\_8\_2/version/latest)

#### Parties Subgraph

[https://api.studio.thegraph.com/query/62454/parties\_base\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/parties\_base\_8\_2/version/latest)

#### Funding Rate Subgraph

[https://api.studio.thegraph.com/query/62454/fundingrate\_base\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/fundingrate\_base\_8\_2/version/latest)

#### Events Subgraph

[https://api.studio.thegraph.com/query/62454/events\_base\_8\_2/version/latest](https://api.studio.thegraph.com/query/62454/events\_base\_8\_2/version/latest)
