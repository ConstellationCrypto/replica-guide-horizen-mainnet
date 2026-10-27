# Horizen Mainnet Replica Guide

1. Set `L1_RPC_URL` in `up.sh`
2. Run `make up`.

## Stack

- **op-reth** `us-docker.pkg.dev/oplabs-tools-artifacts/images/op-reth:v2.6.1`
- **op-node** `us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.19.11`

op-node runs `--syncmode=execution-layer`. op-reth syncs blocks over EL P2P from the trusted peers below (`--disable-discovery` plus `--trusted-peers`), and uses the public sequencer RPC as `--rollup.historicalrpc`. op-node still gossips unsafe payloads over CL P2P to `44.255.138.48:9003`.

## EL P2P

op-reth syncs over EL P2P (devp2p, port `30303` tcp/udp) from Caldera's Horizen replica peers. Discovery is off; op-reth dials these trusted peers directly:

```
enode://ca965bbc0cfce267ed50a8b03dd394a448f39ff234810f1256044f6825a5b5b4b85ef258914bd6671217e8efcae54ed38d73a52d439f860989172c625d16d492@52.13.24.214:30303
enode://ca965bbc0cfce267ed50a8b03dd394a448f39ff234810f1256044f6825a5b5b4b85ef258914bd6671217e8efcae54ed38d73a52d439f860989172c625d16d492@32.184.93.244:30303
enode://ca965bbc0cfce267ed50a8b03dd394a448f39ff234810f1256044f6825a5b5b4b85ef258914bd6671217e8efcae54ed38d73a52d439f860989172c625d16d492@32.185.207.253:30303
```

Outbound TCP to these IPs on `30303` must be allowed. Check that op-reth has peers:

```bash
curl http://localhost:8545 -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}'
```

A result of `"0x0"` means op-reth cannot reach the peers, so EL sync will not progress.

To query the RPC:

```
RPC_URL=http://localhost:8545
curl $RPC_URL -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

To check on the sync status of the node:

```
RPC_URL=http://localhost:7545
curl $RPC_URL -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"optimism_syncStatus","params":[],"id":1}' | jq .
```

or `bash progress.sh`

# Commands:

```
make up
make down
make clean
```


### Snapshot

https://caldera-chain-data-snapshots.s3.us-west-2.amazonaws.com/exported-snapshots/bedrock-horizen/bedrock-horizen-reth-2026-Apr-22.tar