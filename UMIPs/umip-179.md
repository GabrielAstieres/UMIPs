## Headers

| UMIP 179 |  |
| --- | --- |
| UMIP Title | ROPU_Nodeset |
| Authors | Gabriel Astieres |
| Status | Draft |
| Created | 05/10/23 |
| Discourse Link |  |

# Summary

The DVM should support price requests for ROPU_Nodeset. ROPU_Nodeset reflects the amount of staked ETH on the beacon chain by Constellation, a set of minipool validators managed by Nodeset.

# Motivation

Approving this price identifier will allow Rated to serve data on chain to Nodeset. This data will reflect the amount of ETH staked on the beacon chain, allowing the protocol to rebase.

# Data Specifications

The data to refer to when determining the validity of a report is the blockchain itself. In order to access it, one can use a node they have access to or review popular block explorers.

- Execution Layer data: read the Constellation smart contract to determine the validator set.
- Beacon Chain data: gather the balances of relevant validators.

# Price Feed Implementation

No price feed implementation is possible for ROPU_Nodeset.

## Ancillary Data Specifications

The claim made on the Optimistic Oracle V3 is made of bytes containing the total balance of Constellation and the beacon chain epoch at which the measurement was made. The balance being represented by a 128 unsigned integer and the epoch number a 64 unsigned integer.

```solidity
    uint128 balance;
    uint64 epochNumber;
```

# Rationale

The balance of Constellation is reported on a 24-hour cadence. Each of those daily reports will be the object of a claim on the Optimistic Oracle. When a new claim is created, a bytes concatenation of `balance + epochNumber` is included in the claim field. The Oracle is then responsible for verifying that the set of validators forming the constellation has the corresponding amount of staked ETH at the end of the specified epoch.

# Implementation

In order for voters to determine Constellation’s set of validators, they should refer to the `OperatorDistributor` contract of Nodeset deployed at [PLACEHOLDER]. The contract emits events, `MinipoolDestroyed` & `MinipoolCreated`, when a minipool joins or leaves the set. Based on those events one can track the set of validators that form Constellation.

Then, either by accessing a node directly or network explorer (such as: https://www.rated.network/), one can compute the total balance of Constellation by summing the individual balance of each validators.

# Security Considerations

When resolving a dispute, the process in the DVM shall ensure:

- (i) The balance aggregation was computed at the beginning of the declared epoch.
- (ii) The determination of the validator set considers changes that occurred up until the specified epoch.
- (iii) The balance reported has a margin of error of `1_000 GWEI` (`0.000001 ETH`). If voter's verifications find values within the margin, the claim should resolve as true.
