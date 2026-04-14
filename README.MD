# Provably Random On-Chain Raffle

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Solidity](https://img.shields.io/badge/Solidity-0.8.19-blue)](https://soliditylang.org/)
[![Foundry](https://img.shields.io/badge/Built%20with-Foundry-FF6C37)](https://getfoundry.sh/)

A trustless, fully automated lottery smart contract built with Foundry. Winners are selected using **Chainlink VRF v2** for verifiable randomness and **Chainlink Automation** for time-based, gas-efficient upkeep — no centralized server required.

---

## Table of Contents

- [Provably Random On-Chain Raffle](#provably-random-on-chain-raffle)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [How It Works](#how-it-works)
  - [Contract Architecture](#contract-architecture)
    - [Key Functions](#key-functions)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Installation](#installation)
  - [Usage](#usage)
    - [Local Development](#local-development)
    - [Deploy to Sepolia](#deploy-to-sepolia)
    - [Chainlink Subscription Management](#chainlink-subscription-management)
  - [Testing](#testing)
    - [Test Coverage](#test-coverage)
  - [Security](#security)
  - [License](#license)

---

## Overview

| Property | Value |
|---|---|
| Entrance Fee | 0.01 ETH |
| Draw Interval | 30 seconds (configurable) |
| Randomness | Chainlink VRF v2 |
| Automation | Chainlink Automation |
| Network Support | Sepolia Testnet, Anvil (local) |

---

## How It Works

1. **Enter** — A player calls `enterRaffle()` and sends at least the entrance fee in ETH.
2. **Wait** — The raffle remains open until the configured time interval elapses.
3. **Upkeep** — Chainlink Automation calls `checkUpkeep()` on a schedule. When all conditions are met (interval passed, raffle open, balance > 0, players present), it triggers `performUpkeep()`.
4. **Random Selection** — `performUpkeep()` requests a verifiable random number from Chainlink VRF v2. The raffle state moves to `CALCULATING`, blocking new entrants.
5. **Winner Paid** — `fulfillRandomWords()` receives the random number, selects a winner, resets state, and transfers the entire contract balance to the winner automatically.

---

## Contract Architecture

```
src/
└── Raffle.sol              # Core lottery contract

script/
├── DeployRaffle.s.sol      # Deployment script
├── HelperConfig.s.sol      # Network-specific configuration
└── Interactions.s.sol      # Chainlink subscription helpers

test/
├── uint/
│   └── RaffleTest.t.sol    # Unit & fuzz tests
├── integration/
│   └── InteractionsTest.t.sol
└── mocks/
    └── LinkToken.sol       # LINK token mock for local testing
```

### Key Functions

| Function | Visibility | Description |
|---|---|---|
| `enterRaffle()` | `external payable` | Enter the lottery by sending ETH ≥ entrance fee |
| `checkUpkeep()` | `public view` | Called by Chainlink Automation to determine if a draw is needed |
| `performUpkeep()` | `external` | Initiates the VRF random number request |
| `fulfillRandomWords()` | `internal` | VRF callback — selects winner and distributes prize |
| `getEntranceFee()` | `external view` | Returns the current entrance fee |
| `getRaffleState()` | `external view` | Returns `OPEN` or `CALCULATING` |

---

## Getting Started

### Prerequisites

- [Git](https://git-scm.com/)
- [Foundry](https://getfoundry.sh/) — install via:
  ```bash
  curl -L https://foundry.paradigm.xyz | bash && foundryup
  ```

### Installation

```bash
git clone https://github.com/your-username/foundry-smart-contract-lottery-f23
cd foundry-smart-contract-lottery-f23
make install
```

Copy the environment template and fill in your values:

```bash
cp .env.example .env
```

**.env variables required for Sepolia:**

```env
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY
PRIVATE_KEY=0xYOUR_PRIVATE_KEY
ETHERSCAN_API_KEY=YOUR_ETHERSCAN_API_KEY
```

> **Never commit your `.env` file.** It is gitignored by default.

---

## Usage

### Local Development

Start a local Anvil node with deterministic accounts:

```bash
make anvil
```

Deploy to local Anvil:

```bash
make deploy
```

### Deploy to Sepolia

```bash
make deploy ARGS="--network sepolia"
```

This will deploy, verify the contract on Etherscan, and print the deployed address.

### Chainlink Subscription Management

After deploying on Sepolia, fund your VRF subscription and register the consumer:

```bash
# Create a new Chainlink VRF subscription
make createSubscription ARGS="--network sepolia"

# Fund the subscription with LINK
make fundSubscription ARGS="--network sepolia"

# Add the deployed Raffle as a VRF consumer
make addConsumer ARGS="--network sepolia"
```

> You can also manage subscriptions at [vrf.chain.link](https://vrf.chain.link).

---

## Testing

```bash
# Run all tests
make test

# Run tests with verbosity
forge test -vvvv

# Generate a gas snapshot
make snapshot

# Generate coverage report
forge coverage
```

### Test Coverage

Tests cover:

- Raffle initializes in `OPEN` state
- Reverts when entrance fee is insufficient
- Records players on entry
- Emits `EnteredRaffle` event
- Blocks entry during `CALCULATING` state
- `checkUpkeep` returns false when no balance, no players, raffle not open, or interval not passed
- `performUpkeep` reverts when upkeep not needed
- Full end-to-end winner selection with mocked VRF callback

---

## Security

- Uses **CEI pattern** (Checks → Effects → Interactions) throughout.
- Raffle is locked in `CALCULATING` state while awaiting randomness — no re-entrancy window.
- ETH transfer uses low-level `.call` with a failure revert (`Raffle__TransferFailed`).
- No owner-privileged functions; the contract is fully permissionless once deployed.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

[![Michael Vargas Linkedin](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/michael-vargas-a5b51b223/)

*Built following the [Cyfrin Foundry Full Course](https://github.com/Cyfrin/foundry-full-course-f23) by Patrick Collins.*
