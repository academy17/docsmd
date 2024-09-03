# LibMuon

## LibMuon

The Muon Library (`LibMuon`) is an essential component of the SYMM ecosystem, designed to enhance the security and integrity of transactions within the platform. This library utilizes cryptographic functions to verify transactions through multi-signature and gateway checks, leveraging both Threshold Signature Scheme (TSS) and conventional ECDSA signatures. The library's primary function is to ensure that all operations, particularly those involving asset pricing and unrealized profit and loss (uPNL) calculations, are authenticated and verified using trusted and reliable methods.&#x20;

### getChainId()

```solidity
    function getChainId() internal view returns (uint256 id) {
        assembly {
            id := chainid()
        }
    }
```

This function retrieves the unique identifier of the blockchain network (chain ID) on which the smart contract is running. It is essential for verifying the network-specific data and ensuring that the contract operations adhere to the correct blockchain context.

### verifyTSSAndGateway()

The `verifyTSSAndGateway` function in the Muon library is crucial for ensuring the authenticity and integrity of transactions. This function performs a dual verification: one for a threshold signature scheme (TSS) and the other for a gateway signature. Here’s a detailed breakdown of its implementation:

```solidity
    function verifyTSSAndGateway(
        bytes32 hash,
        SchnorrSign memory sign,
        bytes memory gatewaySignature
    ) internal view {
        // == SignatureCheck( ==
        bool verified = LibMuonV04ClientBase.muonVerify(
            uint256(hash),
            sign,
            MuonStorage.layout().muonPublicKey
        );
        require(verified, "LibMuon: TSS not verified");

        hash = hash.toEthSignedMessageHash();
        address gatewaySignatureSigner = hash.recover(gatewaySignature);

        require(
            gatewaySignatureSigner == MuonStorage.layout().validGateway,
            "LibMuon: Gateway is not valid"
        );
        // == ) ==
    }

```

This function ensures that the data has been signed off by the approved parties, providing a reliable trust mechanism within decentralized frameworks.

**TSS Verification:**

* Converts the `hash` of the data into a `uint256` to match the expected input type for the Muon network's verification function.
* Calls `LibMuonV04ClientBase.muonVerify` with the `hash`, `sign` (which contains the signature details), and the public key retrieved from `MuonStorage`.
* Ensures the returned result is `true`, indicating that the TSS signature is valid.

**Gateway Signature Verification:**

* Converts the `hash` into an Ethereum signed message hash to prepare it for signature recovery.
* Recovers the signer's address from the `gatewaySignature` using `hash.recover()`.
* Checks that the recovered address matches the `validGateway` address stored in `MuonStorage`, ensuring the signature is from a trusted source.

In summary, the `verifyTSSAndGateway` function is integral to maintaining the security and reliability of the Symmio ecosystem, ensuring that all transaction-related data is verified through cryptographic methods before any action is taken on the blockchain.

### verifyLiquidationSig()

The `verifyLiquidationSig` function in the Muon library is designed to validate the authenticity and integrity of liquidations. Below is a detailed breakdown of its implementation and functionality:

```solidity
    function verifyLiquidationSig(LiquidationSig memory liquidationSig, address partyA) internal view {
        MuonStorage.Layout storage muonLayout = MuonStorage.layout();
        require(liquidationSig.prices.length == liquidationSig.symbolIds.length, "LibMuon: Invalid length");
        bytes32 hash = keccak256(
            abi.encodePacked(
                muonLayout.muonAppId,
                liquidationSig.reqId,
                liquidationSig.liquidationId,
                address(this),
                "verifyLiquidationSig",
                partyA,
                AccountStorage.layout().partyANonces[partyA],
                liquidationSig.upnl,
                liquidationSig.totalUnrealizedLoss,
                liquidationSig.symbolIds,
                liquidationSig.prices,
                liquidationSig.timestamp,
                getChainId()
            )
        );
        verifyTSSAndGateway(hash, liquidationSig.sigs, liquidationSig.gatewaySignature);
    }
```

This function ensures that the liquidation signal associated with a specific party (partyA) is authentic and hasn't been tampered with. It verifies a set of data including unique identifiers, transaction details, and user-specific nonces against a signature provided via the Muon network.

**Parameter Validation:**

* Ensures that the arrays for `prices` and `symbolIds` within `liquidationSig` have the same length, confirming data consistency and correctness.

**Hash Construction:**

* Constructs a hash from various elements including the Muon app identifier, request ID, liquidation ID, contract address, method identifier, target party's address, nonce for the party, unrealized profit and loss (uPnl), total unrealized loss, symbol IDs, prices, timestamp, and the blockchain's chain ID.
* This hash represents all critical information about the liquidation event that needs to be verified.

**Signature Verification:**

* Calls `verifyTSSAndGateway`, passing the constructed hash along with the signature details (`sigs`) and `gatewaySignature` from `liquidationSig`.
* This step utilizes the Muon network's threshold signature scheme to ensure the liquidation request is indeed signed by the appropriate gateway, adding an additional layer of security and trust.

### verifyQuotePrices()

The `verifyQuotePrices` function in the Muon library plays a critical role in ensuring the integrity and authenticity of price signals for quotes within the Symmio platform.&#x20;

```solidity
    function verifyQuotePrices(QuotePriceSig memory priceSig) internal view {
        MuonStorage.Layout storage muonLayout = MuonStorage.layout();
        require(priceSig.prices.length == priceSig.quoteIds.length, "LibMuon: Invalid length");
        bytes32 hash = keccak256(
            abi.encodePacked(
                muonLayout.muonAppId,
                priceSig.reqId,
                address(this),
                priceSig.quoteIds,
                priceSig.prices,
                priceSig.timestamp,
                getChainId()
            )
        );
        verifyTSSAndGateway(hash, priceSig.sigs, priceSig.gatewaySignature);
    }
```

This function is tasked with validating that the price data associated with specific quote identifiers are legitimate and confirmed by the appropriate Muon gateway.&#x20;

**Parameter Consistency Check:**

* Confirms that the length of the `prices` array matches the length of the `quoteIds` array in the `priceSig` argument. This ensures that each quote ID has a corresponding price, which is fundamental for reliable data processing.

**Hash Construction:**

* Composes a hash from several crucial elements:
  * The Muon app identifier to ensure that the data pertains to the correct application within the Muon network.
  * The unique request ID, which helps in tracking and validating the request.
  * The smart contract address.
  * The list of quote IDs and their corresponding prices
  * The timestamp to contextualize the timing of the data.
  * The chain ID.
* This hash uniquely represents the request for price verification.

**Signature Verification:**

Utilizes the `verifyTSSAndGateway` function to confirm the validity of the data against a threshold signature scheme (TSS) and a gateway signature provided in the `priceSig` structure.

### verifyPartyAUpnlAndPrice

The `verifyPartyAUpnl` function authenticates the unrealized profit and loss (uPnl) data for partyA as well as the price of a symbol, ensuring it originates from a trusted source and is within the valid timeframe. Below is a detailed breakdown of this function's operation within the Symmio platform's context.

```solidity
    function verifyPartyAUpnlAndPrice(
        SingleUpnlAndPriceSig memory upnlSig,
        address partyA,
        uint256 symbolId
    ) internal view {
        MuonStorage.Layout storage muonLayout = MuonStorage.layout();
        // == SignatureCheck( ==
        require(
            block.timestamp <= upnlSig.timestamp + muonLayout.upnlValidTime,
            "LibMuon: Expired signature"
        );
        // == ) ==
        bytes32 hash = keccak256(
            abi.encodePacked(
                muonLayout.muonAppId,
                upnlSig.reqId,
                address(this),
                partyA,
                AccountStorage.layout().partyANonces[partyA],
                upnlSig.upnl,
                symbolId,
                upnlSig.price,
                upnlSig.timestamp,
                getChainId()
            )
        );
        verifyTSSAndGateway(hash, upnlSig.sigs, upnlSig.gatewaySignature);
    }

```

**Implementation Details:**

**Signature Expiry Check:**

* Asserts that the current timestamp is within the valid time range from the signature's timestamp, determined by `upnlValidTime` from the Muon storage settings.&#x20;

**Hash Construction:**

* Constructs a cryptographic hash using key elements which uniquely identify the uPnl data for partyA:
  * Muon application identifier (`muonAppId`) .
  * Request ID (`reqId`) for tracking and validation purposes.
  * Address of the smart contract.
  * Address of partyA, ensuring the data is relevant to the correct party.
  * The nonce associated with partyA to prevent replay attacks.
  * The uPnl value, which is the data subject to verification.
  * The price of the asset in the `upnlSig`.
  * The timestamp.
  * The chain ID.

**Signature Verification:**

* Calls the `verifyTSSAndGateway` function, passing the constructed hash along with the TSS signature and the gateway signature. This step is crucial to verify that the signature:

### verifyPartyBUpnl

The `verifyPartyBUpnl` function in the Muon library authenticates and validate the unrealized profit and loss (uPnl) data for partyB.

```solidity
    function verifyPartyBUpnl(
        SingleUpnlSig memory upnlSig,
        address partyB,
        address partyA
    ) internal view {
        MuonStorage.Layout storage muonLayout = MuonStorage.layout();
        // == SignatureCheck( ==
        require(
            block.timestamp <= upnlSig.timestamp + muonLayout.upnlValidTime,
            "LibMuon: Expired signature"
        );
        // == ) ==
        bytes32 hash = keccak256(
            abi.encodePacked(
                muonLayout.muonAppId,
                upnlSig.reqId,
                address(this),
                partyB,
                partyA,
                AccountStorage.layout().partyBNonces[partyB][partyA],
                upnlSig.upnl,
                upnlSig.timestamp,
                getChainId()
            )
        );
        verifyTSSAndGateway(hash, upnlSig.sigs, upnlSig.gatewaySignature);
    }

```

This function safeguards the uPnl data related to partyB by verifying it against a signed message from a recognized Muon gateway. It ensures that the uPnl data is both current and accurately signed.

**Implementation Details:**

**Hash Composition:**

* Constructs a hash using various key components that uniquely define the uPnl data for partyB:
  * `muonAppId` ensures the context of the application.
  * `reqId` serves as a unique identifier for tracking and validation.
  * The contract's address (`address.this`).
  * `partyB` and `partyA` addresses.
  * Nonce for partyB relative to partyA to prevent replay attacks.
  * The uPnl value, which is the actual data under verification.
  * The timestamp
  * The chainId.

**Signature Verification Process:**

Invokes `verifyTSSAndGateway`, supplying the hash, TSS signature, and gateway signature for validation.

### verifyPairUpnlAndPrice

The `verifyPairUpnlAndPrice` function verifies unrealized profit and loss (uPnl) data along with price information for both partyA and partyB within a trading pair.&#x20;

```solidity
    function verifyPairUpnlAndPrice(
        PairUpnlAndPriceSig memory upnlSig,
        address partyB,
        address partyA,
        uint256 symbolId
    ) internal view {
        MuonStorage.Layout storage muonLayout = MuonStorage.layout();
        // == SignatureCheck( ==
        require(
            block.timestamp <= upnlSig.timestamp + muonLayout.upnlValidTime,
            "LibMuon: Expired signature"
        );
        // == ) ==
        bytes32 hash = keccak256(
            abi.encodePacked(
                muonLayout.muonAppId,
                upnlSig.reqId,
                address(this),
                partyB,
                partyA,
                AccountStorage.layout().partyBNonces[partyB][partyA],
                AccountStorage.layout().partyANonces[partyA],
                upnlSig.upnlPartyB,
                upnlSig.upnlPartyA,
                symbolId,
                upnlSig.price,
                upnlSig.timestamp,
                getChainId()
            )
        );
        verifyTSSAndGateway(hash, upnlSig.sigs, upnlSig.gatewaySignature);
    }
```

This function is designed to validate the uPnl for both parties involved in a trade and the price of the asset at the time the uPnl was recorded.

**Implementation Details:**

**Hash Construction:**

* Constructs a deterministic hash using parameters that uniquely describe the financial state and context of the trading pair:
  * `muonAppId` to contextualize the operation within the Muon network.
  * `reqId` for tracking and verification purposes.
  * The contract's address.
  * Addresses of both partyB and partyA.
  * Nonces for both parties to prevent replay attacks.
  * The uPnl values for both parties.
  * `symbolId` and the associated `price` .
  * The timestamp.
  * The ChainId.

**Signature Verification:**

Calls `verifyTSSAndGateway` with the constructed hash, the TSS signature provided in `upnlSig`, and the gateway signature.
