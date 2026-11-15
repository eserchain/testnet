# Running an Eser Chain node

This document is for the **public testnet** (chain ID 20261113). Everything below was run on the testnet or on the
local test network (Ubuntu 24.04, x86-64) unless it is marked **not verified**. Testnet coins have no value and the
testnet may be restarted with a new genesis. Mainnet (chain ID 19811113) is not launched.

## Network facts

| | |
|---|---|
| Consensus | Proof of Work, original Ethash (no ECIP-1099 / Etchash) |
| Client | Core-Geth `v1.13.0` (github.com/ethereumclassic/core-geth), commit `85093c8531f4a3028244e27f9f15a858737f7ab5` |
| Chain ID / network ID (testnet) | `20261113` (`0x13528f9`) |
| Coin | ESER, 18 decimals |
| Block reward | 2 ESER; +1/32 per included uncle; uncle miner gets (8 − distance)/8 × 2 ESER |
| Block time | about 13 s at equilibrium (Ethash difficulty rule with EIP-100; measured 10.0 s mean while difficulty was still rising) |
| EVM | Shanghai-level opcodes (`PUSH0`), no EIP-1559: no `BASEFEE`; legacy (type 0) transactions tested, access-list (type 1) accepted by the node (not yet seen mined in a block); EIP-1559 (type 2) is rejected |
| Not supported | Cancun opcodes (`MCOPY`, `TLOAD`/`TSTORE`, `BLOBHASH`) |
| Genesis hash (testnet) | `0xba3b827401083284306d2cb4a5f87dd42456387bf6dec7f6e60d0dd771ecc4c6` |
| Bootnodes (testnet) | `enode://bc2b63cb93c0e47eea82776164874b8f83f476043f5d28b61443e2de097697d17cc42111223a300b8af010076740f6b5dfac73606d17fb5a619eee97d982cac1@89.167.127.16:30303`<br>`enode://f3d661e5cd3bacda2cf39b53a4f45ed3f317264fe4c8cd4406554a2b570ee6377702d6f054652c0bf575616b97b3537f9f95e4374bffff51df6737d1a63dd863@2.29.148.35:30303`<br>`enode://ddddf11f5d680ba1849617bee2ded7745759772692bad2901eda56589dd15637e17c941e4dcf7d67f2fc934d84fa39a3194b58da118265d40c7b830b4eb8703c@85.105.194.210:30310`<br>`enode://a9aa2d77f5800cbb73034bf021ed0d93402360098343f676b93dcd5a8e8dd20456bcf79416cca894b43da0e606637be630f4aad2a70b874d6c2a07e274af62d1@37.130.64.31:30303` |
| Explorer | https://explorer.testnet.eserchain.org |
| Pool | `pool.testnet.eserchain.org:8008` (PPLNS, 1 % fee, automatic payouts from 0.2 ESER), page: https://pool.testnet.eserchain.org |
| Public RPC | `https://testnet.eserchain.org/rpc` (rate-limited; for heavy use run your own node) |

Testnet and mainnet will use different genesis files and chain data. Never point a mainnet data directory at a testnet
genesis or the other way round.

## 1. Requirements

- 64-bit Linux (tested: Ubuntu 24.04, x86-64). Windows and macOS builds of Core-Geth exist but are **not verified** for this network.
- 2 CPU cores, 4 GB RAM, 20 GB disk are enough while the chain is young.
- TCP and UDP port 30303 reachable from the internet if you want other nodes to connect to you.
- Mining needs a GPU with more than 1 GB of free memory for the Ethash DAG (1 GB at epoch 0, growing about 8 MB per
  30 000 blocks) and an OpenCL driver.

## 2. Install Core-Geth

Use the official release `v1.13.0` from https://github.com/ethereumclassic/core-geth/releases/tag/v1.13.0 and check
its SHA-256 file before installing.

```bash
VERSION=v1.13.0
BASE=https://github.com/ethereumclassic/core-geth/releases/download/$VERSION
FILE=core-geth-linux-$VERSION.zip
curl -fLO "$BASE/$FILE"
curl -fLO "$BASE/$FILE.sha256"
sha256sum -c "$FILE.sha256"
unzip "$FILE" -d core-geth
sudo install -m 0755 core-geth/geth /usr/local/bin/geth
geth version
```

`geth version` must print `Core-Geth 1.13.0-stable` and `Git Commit: 85093c8531f4a3028244e27f9f15a858737f7ab5`.
The local network was installed from `core-geth-alltools-linux-v1.13.0.zip` (the same `geth` plus `bootnode` and other
tools). The release also has `win64`, `osx`, `arm64` and Docker packages; those are **not verified** on this network.

## 3. Create the data directory

Take `genesis-testnet.json` (next to this document; SHA-256
`45f69c7bf8e34d90f638d744991eaa88c82a02c40c460156cb54a6d13bd45651`) and initialise an empty data directory.
`geth init` only writes the genesis block; it does not delete an existing chain, and it fails if the directory already
holds a different genesis.

```bash
geth init --datadir ~/eser-node genesis-testnet.json
```

The output must show the genesis hash from the table above. If the hash differs, stop: the node would be on a
different chain.

## 4. Start the node

```bash
geth --datadir ~/eser-node --networkid 20261113 --syncmode full \
  --port 30303 --bootnodes "enode://bc2b63cb93c0e47eea82776164874b8f83f476043f5d28b61443e2de097697d17cc42111223a300b8af010076740f6b5dfac73606d17fb5a619eee97d982cac1@89.167.127.16:30303,enode://f3d661e5cd3bacda2cf39b53a4f45ed3f317264fe4c8cd4406554a2b570ee6377702d6f054652c0bf575616b97b3537f9f95e4374bffff51df6737d1a63dd863@2.29.148.35:30303,enode://ddddf11f5d680ba1849617bee2ded7745759772692bad2901eda56589dd15637e17c941e4dcf7d67f2fc934d84fa39a3194b58da118265d40c7b830b4eb8703c@85.105.194.210:30310,enode://a9aa2d77f5800cbb73034bf021ed0d93402360098343f676b93dcd5a8e8dd20456bcf79416cca894b43da0e606637be630f4aad2a70b874d6c2a07e274af62d1@37.130.64.31:30303" \
  --http --http.addr 127.0.0.1 --http.port 8545 --http.api eth,net,web3
```

- `--networkid 20261113` is required; without it the node looks for Ethereum mainnet peers.
- `--syncmode full` is the mode that was tested. Snap sync is **not verified**.
- Optional, recommended: `--mess.activate 3700` turns on MESS (ECBP-1100, protection against deep reorganisations)
  from block 3700, as on the project's own testnet nodes since 13 November 2026 and on mainnet from block 0. It only
  changes which chain your node prefers in a deep reorganisation; it does not change block validity.
- Keep `--http.addr 127.0.0.1`. An RPC port open to the internet lets anyone read your node and, if an account is
  unlocked, spend from it. Do not use `--http.addr 0.0.0.0`, `--allow-insecure-unlock` or the `personal`, `admin`,
  `debug` or `miner` APIs on a reachable address. To use the RPC from another machine, use an SSH tunnel:
  `ssh -N -L 8545:127.0.0.1:8545 user@your-node`.

Check that the node is connected and following the chain:

```bash
geth attach --exec '[net.peerCount, eth.blockNumber, eth.syncing]' ~/eser-node/geth.ipc
```

Stop the node with Ctrl+C or `kill -INT`, so that it closes its database cleanly.

## 5. Wallet (MetaMask)

Add a custom network: RPC URL `https://testnet.eserchain.org/rpc` or your own node (`http://127.0.0.1:8545`, through the
tunnel if it is remote), chain ID
`20261113`, currency symbol `ESER`. Coin transfers and contract calls from MetaMask were tested.

Your private key and recovery phrase stay in your wallet. No node operator, pool or support channel needs them.

## 6. Mining

### Solo, with a GPU

Run the node with the miner serving work but not sealing on the CPU, and point a GPU miner at it:

```bash
geth ... --mine --miner.threads -1 --miner.etherbase 0xYOUR_ADDRESS
ethminer -G -P http://127.0.0.1:8545
```

`--miner.etherbase` is the address that receives the rewards; only the address is needed, the node needs no key.

### Through a pool

```bash
ethminer -G -P stratum1+tcp://0xYOUR_ADDRESS.WORKER@pool.testnet.eserchain.org:8008
```

`stratum1` (eth-proxy) and `stratum2` (EthereumStratum/1.0.0, NiceHash style) both work with the Eser pool.
Pool page with your balance and payments: https://pool.testnet.eserchain.org

### Miner software

| Miner | Result |
|---|---|
| ethminer 0.19.0-18 (commit `8eaf50ab`, built from source; Linux x86-64 build on https://eserchain.org/dl/), OpenCL (`-G`) | Works. RTX 5060 Ti: 53.3 MH/s |
| ethminer 0.18.0 (official release), OpenCL | Works on 16 GB cards. Fails on a 4 GB NVIDIA card (`CL_INVALID_BUFFER_SIZE`): it allocates the DAG as one buffer |
| ethminer 0.19.0-18 (commit `8eaf50ab`), Windows build made for this project (https://eserchain.org/dl/), OpenCL (`-G`) | Works on a 4 GB card: RTX 3050 Ti Laptop GPU, split DAG, about 22 MH/s, 1189 shares accepted and none rejected in a long run on the testnet pool |
| ethminer 0.18.0 / 0.19.0-alpha, CUDA (`-U`) | Does not work on RTX 30 series and newer (`invalid device symbol`) |
| lolMiner 1.98a (closed source, 0.7 % dev fee), CUDA on NVIDIA | Works. RTX 5060 Ti: 52.45 MH/s, 136 of 136 shares accepted. `lolMiner --algo ETHASH --pool pool.testnet.eserchain.org:8008 --user 0xYOUR_ADDRESS.RIG --ethstratum ETHPROXY` |
| GMiner 3.44 (closed source, 2 % dev fee) | Does not run on RTX 50 series (`no kernel image is available for execution on the device`). Older cards **not verified** |
| T-Rex 0.26.8 (closed source, last release 2022) | Does not run on RTX 50 series (`Can't start T-Rex, cuda_crc32_tweaked_120.cubin.c: No such file or directory`). Older cards **not verified** |
| AMD R9 380 (Tonga, 2 GB), Ubuntu 24.04, ethminer 0.19.0-18 (project Linux build) | Works only with AMD's legacy OpenCL (`opencl-legacy-amdgpu-pro-icd` 23.20 and `libdrm-amdgpu-amdgpu1` from repo.radeon.com/amdgpu/5.7.1, files extracted, ICD in `/etc/OpenCL/vendors/`, `LD_LIBRARY_PATH=/opt/amdgpu/lib/x86_64-linux-gnu`): about 16.6 MH/s, 40 of 40 shares accepted in the first test. Mesa Rusticl aborts while building the kernel; Mesa Clover gives invalid results; lolMiner 1.98a rejects the card |
| Other AMD cards, other miners | **Not verified** |
| ASIC miners | **Not verified** |

The miner must use plain Ethash. Etchash (Ethereum Classic) uses longer DAG epochs, so a miner set to Etchash is
expected to produce invalid shares from block 30 000 on (**not verified**: the test chains are still in epoch 0).

CPU mining (`geth --mine --miner.threads N`) works but is only useful on a test network with very low difficulty.

## 7. Smart contracts

Compile with EVM version `shanghai` (Solidity: `evmVersion: "shanghai"`, Remix: Advanced Configurations → EVM
Version). Newer targets may emit `MCOPY` or transient storage opcodes, which this chain rejects with `invalid opcode`.
`block.basefee` is not available. Deployment and calls were tested with Solidity 0.8.24 through Remix and MetaMask.

To publish the source, open the contract in the explorer (https://explorer.testnet.eserchain.org) → **Contract** →
**Verify & publish**, and give the same compiler version, EVM version (`shanghai`) and optimizer setting you compiled
with. A single-file upload usually gives a *partial* match: the running code is identical, only the metadata hash at
the end of the bytecode differs because Remix compiles the file under another path. For a full match, use the
"Standard JSON input" method with the input JSON from Remix's compilation details. Tested on the testnet with Solidity
0.8.24 (partial match).

## 8. Troubleshooting

| Symptom | Cause |
|---|---|
| `net.peerCount` stays 0 | Wrong `--networkid`, wrong or missing `--bootnodes`, or port 30303 blocked |
| Peers connect and drop at once | Different genesis. Compare the genesis hash printed by `geth init` |
| `Fatal: Failed to write genesis block: database contains incompatible genesis` | The data directory belongs to another chain. Use a new, empty `--datadir`; do not delete a directory you still need |
| Transaction stays pending | No one is mining, or the gas price is below what miners accept (default 1 gwei) |
| `invalid opcode: MCOPY` (or `BASEFEE`, `TLOAD`) | Contract compiled for a newer EVM version; see section 7 |
| Windows: ethminer prints `No OpenCL platforms found` | The NVIDIA OpenCL driver is not registered under `HKLM\SOFTWARE\Khronos\OpenCL\Vendors` |
