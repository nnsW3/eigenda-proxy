# EigenDA Plasma DA Server

## Introduction

This simple DA server implementation supports ephemeral storage via EigenDA. 

## EigenDA Configuration
Additional cli args are provided for targeting an EigenDA network backend:
- `--eigenda-rpc`: RPC host of disperser service. (e.g, on holesky this is `disperser-holesky.eigenda.xyz:443`)
- `--eigenda-status-query-timeout`: (default: 25m) Duration for which a client will wait for a blob to finalize after being sent for dispersal.
- `--eigenda-status-query-retry-interval`: (default: 5s) How often a client will attempt a retry when awaiting network blob finalization. 
- `--eigenda-use-tls`: (default: true) Whether or not to use TLS for grpc communication with disperser.

## Running Locally
1. Compile binary: `make da-server`
2. Run binary; e.g: `./bin/da-server --addr 127.0.0.1 --port 6969 --eigenda-rpc disperser-holesky.eigenda.xyz:443 --eigenda-status-query-timeout 45m`

## Breaking changes from existing OP-Stack

### Server / Client
Unlike the keccak256 DA server implementation where commitments can be generated by the batcher, commitments with EigenDA are only derivable **once** a blob has been successfully finalized (i.e, dispersed, confirmed, and submitted within a batch to Ethereum). The existing `op-plasma` schema in the monorepo of having a precomputed key was broken in the following ways:
* POST `/put` endpoint was modified to remove the `commitment` query param and return the generated `commitment` value in the response body
* Modified `DaClient` to use an alternative request/response flows with server for inserting and fetching preimages

### Commitment Schemas
An `EigenDACommitment` layer type has been added that supports verification against its respective pre-images. Otherwise this logic is pseudo-identical to the existing `Keccak256` commitment type. The commitment is encoded via the following byte array:
```
            0        1        2        3        4                 N
            |--------|--------|--------|--------|-----------------|
             commit   da layer  ext da   version  raw commitment
             type       type    type      byte

```

## Testing
Some unit tests have been introduced to assert correctness of encoding/decoding logic. These can be ran via `make test`.

Otherwise a single E2E test (`test/e2e_test.go`) exists which asserts that a commitment can be generated when inserting some arbitrary data to the server and can be read using the commitment for a key lookup. This can be ran via `make e2e-test`. Please **note** that this test uses the EigenDA Holesky network which is subject to rate-limiting and slow confirmation times *(i.e, >10 minutes per blob confirmation)*. Please advise EigenDA's [inabox](https://github.com/Layr-Labs/eigenda/tree/master/inabox#readme) if you'd like to spinup a local DA network for faster iteration testing. 


## Resources
- [op-stack](https://github.com/ethereum-optimism/optimism)
- [plasma spec](https://specs.optimism.io/experimental/plasma.html)
- [eigen da](https://github.com/Layr-Labs/eigenda)
