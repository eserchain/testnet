# Eser Chain White Paper

Version 1.0, 13 November 2026. Values marked **TBD** are decided before the mainnet launch.

## Abstract

Eser Chain is an independent, Ethereum-compatible Proof-of-Work blockchain. Blocks are mined with the original Ethash
algorithm, so anyone with a common GPU can take part. The chain runs the Ethereum Virtual Machine: Solidity contracts,
MetaMask and the usual Ethereum tools work unchanged. The native coin is ESER. There is no premine and no sale: every
ESER in existence is created by mining, under rules fixed in the genesis block. The block reward starts at 2 ESER and
halves every 9,700,000 blocks (about four years), which limits the supply from block rewards to about 38.8 million ESER.

## 1. Motivation

Since Ethereum moved to Proof of Stake, GPU owners have few chains where ordinary hardware still produces blocks, and
many new chains start with large allocations to their founders or investors. Eser Chain aims at the opposite:

- **Open mining.** Ethash runs on the GPUs people already own. A 4 GB card mines today; the project publishes miner
  builds for Windows and Linux with their source code.
- **Fair start.** No coins exist before the first block is mined. The founder mines under the same rules as everyone.
- **Known tools.** Developers use Solidity, Remix, MetaMask and the existing Ethereum libraries without changes.
- **Plain rules.** The client is an unmodified release of Core-Geth; the chain's rules are a public genesis file.

## 2. Consensus

| | |
|---|---|
| Algorithm | Ethash, original (not Etchash): DAG epochs of 30,000 blocks; the DAG starts at about 1 GB and grows by about 8 MB per epoch |
| Block time | about 13 seconds |
| Difficulty adjustment | Homestead rule with EIP-100 (uncles counted); difficulty changes by at most 1/2048 per block |
| Fork choice | Heaviest total difficulty, with MESS (ECBP-1100) against deep reorganizations (section 6) |
| Uncle blocks | Up to two per block, at most six generations old |

**Block time.** The difficulty rule compares each block's time with a fixed divisor of 9 seconds. At equilibrium it
settles near 9 / ln 2 ≈ 13 seconds. The target is not a parameter: the stock rule is kept so that the chain runs on an
unmodified client.

**Uncle blocks.** A valid block found at almost the same time as the winning block can be included as an uncle block.
Its miner receives (8 − d) / 8 of the block reward, where d is how many blocks later it is included (1 to 6); the
including miner receives an extra 1/32 of the block reward per uncle block. Uncle rewards lower the cost of network
latency for small and distant miners.

## 3. Execution layer

| | |
|---|---|
| Virtual machine | EVM with the opcodes of Ethereum's Shanghai upgrade (including `PUSH0`), active from block 0 |
| Transactions | Legacy (type 0) and access-list (type 1) transactions, EIP-155 replay protection |
| Fees | Gas price auction; the whole fee goes to the miner. No EIP-1559 base fee, nothing is burnt |
| Not supported | Cancun opcodes (`MCOPY`, `TLOAD`/`TSTORE`, `BLOBHASH`) and `BASEFEE` |
| Gas limit | 30,000,000 at genesis; miners can move it up or down gradually |
| Chain ID | Mainnet `19811113`, testnet `20261113` |

Contracts are compiled for EVM version `shanghai`. Deployment, calls, events and source verification were tested on
the testnet with Solidity 0.8.24, Remix and MetaMask. The block explorer verifies contract sources.

## 4. Monetary policy

The block reward starts at 2 ESER and halves every 9,700,000 blocks. At 13-second blocks one era lasts about four years.

| Era | Blocks | Block reward | Starts after about | Supply from block rewards at the end of the era |
|---|---|---|---|---|
| 0 | 0 – 9,699,999 | 2 ESER | year 0 | 19.40 M |
| 1 | 9,700,000 – 19,399,999 | 1 ESER | year 4 | 29.10 M |
| 2 | 19,400,000 – 29,099,999 | 0.5 ESER | year 8 | 33.95 M |
| 3 | 29,100,000 – 38,799,999 | 0.25 ESER | year 12 | 36.38 M |
| 4 | 38,800,000 – 48,499,999 | 0.125 ESER | year 16 | 37.59 M |
| 5 | 48,500,000 – 58,199,999 | 0.0625 ESER | year 20 | 38.19 M |

The reward reaches zero at the 61st halving. Block rewards create about **38.8 million ESER** in total. Uncle and
nephew rewards add a small, usage-dependent amount on top; transaction fees move existing ESER to miners and create
none. "Years" assume an average block time of 13 seconds; the real calendar depends on the difficulty adjustment.

The schedule is written into the genesis file as 62 activation points (`blockReward`). It was tested on a separate
chain with a short interval: the client paid 2, 1, 0.5, 0.25 and 0.125 ESER at the expected heights. The mining pool
computes rewards with the same schedule.

## 5. Fair launch

- **No premine.** The genesis block allocates no coins to anyone.
- **No sale.** There is no ICO, presale or token sale, and the project does not sell ESER.
- **Same rules for everyone.** The founder mines like any other miner.
- **Genesis message.** The genesis block carries the sentence
  *"Peace at home, peace in the world. M.K. Ataturk"*.
- **Genesis file.** The launch genesis file and its SHA-256 are published. Node operators must check that SHA-256:
  the reward schedule is part of the file but not of the genesis block hash, so a node with a wrong file would run the
  same chain for four years and then split off at the first halving.

## 6. Security

**Majority attacks.** On any Proof-of-Work chain, whoever controls most of the hash power can rewrite recent history,
for example to reverse a payment (a 51 % attack). A young chain with little hash power is exposed to anyone who rents or
owns more. The lasting defence is many independent miners; the measures below reduce the damage until then.

**MESS (ECBP-1100).** Core-Geth's Modified Exponential Subjective Scoring is active from block 0. A node asks a late,
competing chain segment for much more total difficulty than its own, up to about 31 times for segments older than
about seven hours. Short, natural reorganizations are not affected. In a test on a private network, a 45-minute private
chain with 1.56 times the honest chain's total difficulty was published: five MESS nodes rejected it, while a node with
MESS turned off reorganized 154 blocks to follow the attacker.

Limits of MESS: it engages only on fully synced nodes with at least five peers; nodes that are isolated or join during
an attack can be misled; a determined attacker with far more hash power can still win; and when MESS nodes and other
nodes disagree, the network can split until one side falls behind. Exchanges, pools and services should run their own
nodes with enough peers and keep MESS on.

**Confirmations.** Services should wait for more confirmations the larger the amount, and more on a young network. The
mining pool credits rewards only after 32 blocks.

**Operations.** Wallet keys are never shared; public RPC endpoints expose no account functions; pool and faucet wallets
are hot wallets holding only operating amounts.

## 7. Network and software

| Component | Software | Notes |
|---|---|---|
| Node | Core-Geth v1.13.0, official release, unmodified | Linux x86-64 and arm64 tested |
| Bootnodes | Four, run by the project, in three networks (a data centre and two internet lines) | More independent bootnodes are welcome |
| Mining pool | open-etc-pool-friends with project fixes | PPLNS, 1 % fee, automatic payouts from 0.2 ESER |
| Explorer | Blockscout | Contract source verification |
| Faucet | Project service | Testnet only |
| Public RPC | Rate-limited endpoint | Heavy users should run their own node |

**Tested miners.** ethminer 0.19 (project builds, OpenCL, split DAG for 4 GB cards); lolMiner 1.98a; GMiner 3.44 and
T-Rex 0.26.8 on RTX 30 series cards (they do not start on RTX 50 series cards). AMD cards are not tested yet.

All configuration, guides and results of the tests above are public: https://eserchain.org and
https://github.com/eserchain/testnet.

## 8. Governance

Eser Chain is a founder-led project. The founder sets the protocol rules and the roadmap.

- Rule changes are hard forks. They are announced in advance with the activation block number, the reason and the
  code or configuration change, and published in the public repository.
- A change takes effect on the network only when node operators and miners run the new release. No one can force
  a change on nodes that do not upgrade.
- Changes are tested on the testnet before they reach the mainnet.

## 9. Roadmap

| Stage | Status |
|---|---|
| Local test network, pool, explorer, miner builds | Done |
| Public testnet (chain ID 20261113), three bootnodes, pool, explorer, faucet, public RPC | Live since 13 November 2026 |
| Testnet in real use: outside miners and nodes, several weeks without problems | In progress |
| Mainnet preparation: separate infrastructure, independent bootnode locations, pool wallet, start difficulty, security review | Next |
| Mainnet launch (chain ID 19811113) | **TBD** |

## 10. Disclaimer

This paper describes software and network rules. It is not an offer to sell or buy anything, investment advice or a
promise of value. ESER has no price set by the project. Testnet ESER has no value. Mining uses electricity and wears
hardware; blockchains can fail, split or be attacked; software can contain errors. Anyone who mines, runs a node or
holds ESER does so at their own risk. The project will never ask for private keys, recovery phrases or passwords.
