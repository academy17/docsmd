# MuonStorage

The `MuonStorage` contract is designed to help facilitate the verification of data related to trading. It outlines the structure of various signatures and configurations.&#x20;

## **Structs**

### **SchnorrSign**

Represents a Schnorr signature, which is used for cryptographic verification of transactions and data. It includes:

```solidity
struct SchnorrSign {
    uint256 signature;
    address owner;
    address nonce;
}
```

* `signature`: The actual signature.
* `owner`: The address of the owner who generated the signature.
* `nonce`: An additional parameter used to ensure the uniqueness of the signature, preventing replay attacks.

### **PublicKey**

Represents a public key used in cryptographic operations.

```solidity
struct PublicKey {
    uint256 x;
    uint8 parity;
}
```

* `x`: The x-coordinate of the public key on the elliptic curve.
* `parity`: Indicates the parity of the y-coordinate of the public key, which helps in recovering the full public key from the x-coordinate alone.

### **SingleUpnlSig**

Stores the signature for a single unrealized profit and loss (uPnl) computation.

```solidity
struct SingleUpnlSig {
    bytes reqId;
    uint256 timestamp;
    int256 upnl;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

* `reqId`: A unique identifier for the request.
* `timestamp`: The timestamp when the signature was generated.
* `upnl`: The unrealized profit or loss.
* `gatewaySignature`: Additional security layer provided by a gateway signature.
* `sigs`: Embedded Schnorr signature for verification.

### **SingleUpnlAndPriceSig**

Extends `SingleUpnlSig` by adding price information, used for verifying computations involving both uPnl and asset prices.

```solidity
struct SingleUpnlAndPriceSig {
    bytes reqId;
    uint256 timestamp;
    int256 upnl;
    uint256 price;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

* `price`: The price of the asset.

### **PairUpnlSig**

Contains signatures for calculations involving two parties (PartyA and PartyB).

```solidity
struct PairUpnlSig {
    bytes reqId;
    uint256 timestamp;
    int256 upnlPartyA;
    int256 upnlPartyB;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

* `upnlPartyA` and `upnlPartyB`: uPnl values for both parties.
* Includes all other fields from `SingleUpnlSig`.

### **PairUpnlAndPriceSig:**

Combines features of `PairUpnlSig` and `SingleUpnlAndPriceSig` for scenarios where both uPnl calculations and price verifications are necessary for two parties.

```solidity
struct PairUpnlAndPriceSig {
    bytes reqId;
    uint256 timestamp;
    int256 upnlPartyA;
    int256 upnlPartyB;
    uint256 price;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

### **LiquidationSig:**

Signature struct specifically designed for use in liquidation scenarios.

```solidity
struct LiquidationSig {
    bytes reqId;
    uint256 timestamp;
    bytes liquidationId;
    int256 upnl;
    int256 totalUnrealizedLoss; 
    uint256[] symbolIds;
    uint256[] prices;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

* `liquidationId`: Identifier for the specific liquidation event.
* `totalUnrealizedLoss`: Represents the total unrealized loss leading to liquidation.
* `symbolIds` and `prices`: Lists of symbol identifiers and their corresponding prices involved in the liquidation.

### **QuotePriceSig:**

Utilized for verifying the prices of multiple quotes in a single operation.

```solidity
struct QuotePriceSig {
    bytes reqId;
    uint256 timestamp;
    uint256[] quoteIds;
    uint256[] prices;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```

* `quoteIds`: List of quote identifiers.
* `prices`: Corresponding prices for the quotes.
