# The SYMM Diamond

SYMMIO's core contracts are implemented with **EIP-2535**, a solution for building modular and upgradeable contracts.

### **EIP-2535: Diamond Standard Overview**

EIP-2535 introduces the Diamond Standard in smart contract development. This standard centralizes around the concept of a "Diamond" contract, which is constructed from multiple "facets," each responsible for different functionalities. This modular design enhances maintainability by allowing facets to be upgraded or modified independently without affecting the entire contract.

Key to this architecture are function selectors, which serve as unique identifiers for each function within the facets. These selectors efficiently direct function calls to the correct facet, facilitating organized and conflict-free code execution.

For further reading and technical details, refer to the [official EIP-2535 documentation](https://eips.ethereum.org/EIPS/eip-2535).

## SYMMIO's Diamond Implementation

A **diamond** is a facade smart contract that `delegatecall`s into its facets to execute function calls. Facets are separate, independent contracts that can share internal functions, libraries, and state variables. All the data used and manipulated by the facets is stored in the Diamond contract. The facets contain the code that operates on this data, but the state itself is held in the storage of the Diamond contract.

Libraries help define the structure of data and the methods to interact with it. When a facet needs to interact with shared data, it uses the library to define how to access and manipulate this data.

Each of the storage libraries define:

* A `bytes32` storage slot that is the keccak hash of a unique identifier string (e.g. `diamond.standard.storage.quote`)&#x20;
* A `Layout` struct, which organizes and manages state variables. It centralizes state management to ensure that different facets of a contract can access and modify shared data in a consistent manner.

The `layout()` function in each of the storage files uses inline assembly to calculate the exact storage slot where the `Layout` struct is stored using a keccak256 hash of a unique identifier string. It returns a reference to the `Layout` struct, specifically pointing to the designated storage slot. This returned reference allows other parts of the contract to read from and write to the shared state variables defined within the `Layout`.

### Storage Libraries

* `GlobalAppStorage.sol:` Storage related to SYMMIO emergency functions.
* `MAStorage.sol:` This is the Master Agreement Storage. It outlines all the variables related to the conditions of partyB (the solver), such as cooldowns related to the forced canceling of closed positions, liquidator shares, and the liquidation statuses across partyB's
* `QuoteStorage.sol:` This defines the Quote, OrderType, QuoteStatus and PositionType. The Layout Struct contains mappings related to Quotes.
* `MuonStorage.sol:` This defines the structure of signatures required for verifying the price of a symbol, user uPnl and liquidation state.
* `AccountStorage.sol:` This is primarily used to store locked and available balances.
* `SymbolStorage.sol:` Defines a Symbol struct, which contains details about the trading symbol. The Layout contains a mapping of symbols.

### Facet Interaction with Diamond Storage

The Diamond contract itself doesn't need to import these storage libraries because it does not directly use them. When a facet, which imports and uses one of these storage libraries, is invoked via a `delegatecall` from the Diamond, it operates directly on the Diamond’s storage. The facet uses the slot address computed by the library to access or modify the data in the specific storage slot. This operation is transparent to the Diamond itself.

### **`IDiamondCut` Interface**

A diamond contains within it a mapping of function selectors to facet addresses. Functions are added/replaced/removed by modifying this mapping. The SYMM Diamond implements the `IDiamondCut` interface to allow modifications to the function selector mapping. The `diamondCut` function updates any number of functions from any number of facets in a single transaction. Executing all changes within a single transaction prevents data corruption which could occur in upgrades done over multiple transactions.

### Overview of The SYMM Diamond

The diagram below shows the relationship of facets with the Diamond

<figure><img src=".gitbook/assets/Diamond.jpg" alt=""><figcaption><p>Diagram showing the facets and libraries that interact with the core diamond</p></figcaption></figure>

### Finding Facet Contracts

In the SYMM Diamond, the `facets()` function of the `DiamondLoupeFacet` plays a crucial role in inspecting the contract's structure and behaviour. When this function is called on the Diamond, it returns an array of `Facet` structs, each containing the address of a facet along with the specific function selectors associated with that facet.&#x20;

&#x20;For a user-friendly examination of all facets associated with a Diamond contract, consider using the [louper.dev](https://louper.dev/) tool.
