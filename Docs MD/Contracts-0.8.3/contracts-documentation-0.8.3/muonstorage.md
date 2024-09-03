# MuonStorage

## Deferred Liquidation

Previously, the `upnlValidTime` variable in the Muon configuration prevented the liquidator from liquidating a user after a certain amount of time. This update allows the liquidator to liquidate the user if they haven't increased their nonce, provided the Muon app can supply signatures for a specific block requested by the liquidator. The `LiquidationSig` struct has been updated accordingly:

```solidity
// From
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

// To
struct LiquidationSig {
    bytes reqId;
    uint256 timestamp; // Timestamp when the liquidation signature was created
    uint256 liquidationBlockNumber; // Block number at which the user became insolvent
    uint256 liquidationTimestamp; // Timestamp when the user became insolvent
    uint256 liquidationAllocatedBalance; // User's allocated balance at the time of insolvency
    bytes liquidationId;
    int256 upnl; // User's unrealized profit and loss at the time of insolvency
    int256 totalUnrealizedLoss; // Total unrealized loss of the user at the time of insolvency
    uint256[] symbolIds;
    uint256[] prices;
    bytes gatewaySignature;
    SchnorrSign sigs;
}
```
