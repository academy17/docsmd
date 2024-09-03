---
description: This section covers the changes in SYMMIO v0.8.3 from v0.8.2
---

# Contracts Documentation 0.8.3

### General Changes: <a href="#general-changes" id="general-changes"></a>

## Remove unnecessary nonce increase <a href="#remove-unnecessary-nonce-increase" id="remove-unnecessary-nonce-increase"></a>

There were some scenarios before in the contracts where we increased the nonce to make everything more secure. However, upon further consideration, we realized that it is not necessary and makes other processes in the system, like liquidation, more complex. Therefore, we've removed the nonce increase in the following methods:

* `allocate()`
* `deallocate()`
* `transferAllocation()`
* `allocateForPartyB()`
* `deallocateForPartyB()`

### Other Bug Fixed And Enhancements <a href="#other-bug-fixed-and-enhancements" id="other-bug-fixed-and-enhancements"></a>

* Add `notSuspended` modifier to `depositAndAllocate`, `internalTransfer`
* Add `newAllocatedAmount` in `InternalTransfer`, `transferDeallocation` events
* Skip the check in `openPosition` that checks whether the remaining position value is acceptable if the rest is going to be cancelled

[\
](https://docs.symm.io/\~/changes/tjuxT3IXBb1TlxV8aBgR/protocol-architecture/technical-documentation/contracts-documentation-0.8.2/symm-app-muon/muonstorage)
