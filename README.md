# Eser Chain testnet

Eser Chain is an independent, Ethereum-compatible Proof-of-Work blockchain mined with the original Ethash algorithm.
This repository holds what you need to join the **public testnet** and is the place to report problems.

> **Testnet only.** Testnet ESER has no value and the testnet may be restarted. Mainnet is not launched; there is no
> premine, no token sale and nothing to buy. Nobody from this project will ask for your private key, recovery phrase,
> passwords or money.

| | |
|---|---|
| Chain ID / network ID | `20261113` (`0x13528f9`) |
| Consensus | Proof of Work, original Ethash (not Etchash) |
| Block reward / time | 2 ESER / about 13 s |
| Client | [Core-Geth v1.13.0](https://github.com/ethereumclassic/core-geth/releases/tag/v1.13.0) |
| Genesis | [`genesis-testnet.json`](genesis-testnet.json), hash `0xba3b827401083284306d2cb4a5f87dd42456387bf6dec7f6e60d0dd771ecc4c6` |
| Bootnodes | [`bootnodes.txt`](bootnodes.txt) |
| EVM | Shanghai-level opcodes, legacy transactions (no EIP-1559): compile contracts for EVM `shanghai` |
| Web site | https://eserchain.org |
| Explorer | https://explorer.testnet.eserchain.org |
| Pool | `pool.testnet.eserchain.org:8008` · https://pool.testnet.eserchain.org (PPLNS, 1 % fee, payouts from 0.2 ESER) |
| Public RPC | `https://testnet.eserchain.org/rpc` (rate-limited) |
| Faucet | on https://eserchain.org |
| White paper | [WHITEPAPER.md](WHITEPAPER.md) · [Türkçe](WHITEPAPER-tr.md) · [PDF](https://eserchain.org/eser-chain-whitepaper.pdf) |
| Network status | https://eserchain.org/status.html |

## Mine

```
ethminer -G -P stratum1+tcp://0xYOUR_ADDRESS.RIG@pool.testnet.eserchain.org:8008
```

Windows and Linux builds of ethminer 0.19.0-18 (OpenCL, split DAG: 4 GB cards work) with their source are on
https://eserchain.org. Your GPU needs more than 1 GB of free memory.

lolMiner 1.98a also works:

```
lolMiner --algo ETHASH --pool pool.testnet.eserchain.org:8008 --user 0xYOUR_ADDRESS.RIG --ethstratum ETHPROXY
```

GMiner 3.44 and T-Rex 0.26.8 do not run on RTX 50 series cards. Results for other cards and miners: see
[NODE.md](NODE.md); please report yours.

## Run a node

```
geth init --datadir ~/eser-node genesis-testnet.json
geth --datadir ~/eser-node --networkid 20261113 --syncmode full \
  --bootnodes "$(paste -sd, bootnodes.txt)" \
  --http --http.addr 127.0.0.1 --http.api eth,net,web3
```

Full guide, tested miners and troubleshooting: [NODE.md](NODE.md).

## Wallet

MetaMask: add a network with RPC URL `https://testnet.eserchain.org/rpc` (or your own node), chain ID `20261113`,
symbol `ESER`, explorer `https://explorer.testnet.eserchain.org`. Or use the button on https://eserchain.org.

## Report a problem

Open an [issue](../../issues/new/choose). For mining problems give your GPU, driver, operating system, miner and
version, the command you ran and the error lines. Do not post private keys, recovery phrases or passwords.

## License

Documentation (README.md, NODE.md, WHITEPAPER.md, WHITEPAPER-tr.md, issue templates): [CC BY 4.0](LICENSE), © Eser Chain.
Network data (`genesis-testnet.json`, `bootnodes.txt`): public domain, [CC0 1.0](LICENSE-CC0) – copy it anywhere without
attribution. The miner builds on https://eserchain.org are GPL-3.0, like ethminer itself.
