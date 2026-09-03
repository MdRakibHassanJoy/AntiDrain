# 🛡️ AntiDrain — EVM Wallet Asset Recovery

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![EIP-7702](https://img.shields.io/badge/EIP--7702-Prague%20%2F%20Isthmus-green.svg)](https://eips.ethereum.org/EIPS/eip-7702)
[![EIP-712](https://img.shields.io/badge/EIP--712-Typed%20Authorization-orange.svg)](https://eips.ethereum.org/EIPS/eip-712)
[![Manifest V3](https://img.shields.io/badge/Manifest%20V3-Least%20Privilege-blueviolet.svg)](extension/manifest.json)
[![Tests: 504 Passing](https://img.shields.io/badge/Tests-504%20Passing-success.svg)](#testing)

**AntiDrain** is a non-custodial, open-source browser companion and CLI toolkit for rescuing tokens, native assets, and claimable airdrops from compromised Ethereum / EVM wallets — without ever funding the compromised account with gas.

It uses **EIP-7702 sponsored delegation** and **EIP-712 typed intents** so a designated **Sponsor Wallet** pays gas while the smart contract atomically sweeps assets to a verified **Safe Wallet** in a single transaction.

---

## How It Works

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                    WEBPAGE (Untrusted)                                   │
│   Airdrop portals, dApps, hostile scripts                                │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ window.postMessage (EIP-1193)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              CONTENT SCRIPT (Isolated World)                             │
│   Sanitizes requests · Blocks signing oracles (Error 4100)               │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ chrome.runtime.sendMessage
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              BACKGROUND SERVICE WORKER                                   │
│   AES-256-GCM + PBKDF2 encrypted vault · Spending policy guards          │
│   Human confirmation modal · Restricted broadcast transport              │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ eth_sendRawTransaction
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              ON-CHAIN RESCUE CONTRACT                                    │
│   msg.sender == authorized sponsor (cryptographically bound via EIP-712) │
│   ERC-7201 unstructured nonces · EIP-1153 transient reentrancy lock     │
│   Atomic sweep → Immediate sponsored revocation to address(0)            │
└─────────────────────────────────────────────────────────────────────────┘
```

**Recovery flow:**
1. User configures their dedicated, user-funded Sponsor Wallet and Safe Destination Wallet.
2. Compromised victim EOA signs an EIP-7702 delegation tuple to the canonical delegate contract (`0x60BAf255624BEE5629e5D86Ae8976aF19795A314`) and an EIP-712 typed intent explicitly authorizing the sponsor wallet.
3. Sponsor broadcasts a Type 0x04 transaction paying gas. The delegate derives `sponsor = msg.sender`, validates that the victim authorized that exact sponsor, and atomically sweeps 100% of discovered tokens and native funds to the safe wallet.
4. A follow-up sponsored transaction immediately revokes the delegation to `address(0)`.

---

## Supported Networks

| Network | Chain ID | Gas Token | Broadcast Transport |
|:---|:---:|:---:|:---|
| **Ethereum** | `1` | ETH | Private builder relay (Flashbots Protect) |
| **Base** | `8453` | ETH | Direct sequencer submission |
| **Arbitrum One** | `42161` | ETH | Sequencer submission |
| **Optimism** | `10` | ETH | Sequencer submission |
| **Polygon PoS** | `137` | **POL** | RPC submission (no cryptographic MEV privacy) |
| **Mantle** | `5000` | **MNT** | Chain-specific sequencer transport |

All broadcast submissions route strictly to dedicated endpoints with **zero fallback to public RPC aggregators**.

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v20+
- [Foundry](https://getfoundry.sh/) (`forge` / `anvil`)

### 1. Clone & Install

```bash
git clone https://github.com/MdRakibHassanJoy/AntiDrain.git
cd AntiDrain

# CLI
cd cli && npm install && npm run build && cd ..

# Smart Contracts
forge build --root contracts
```

### 2. Build the Browser Extension

```bash
node extension/scripts/package-store-zip.mjs
```

This compiles TypeScript sources and packages the extension for Chrome Web Store submission.

### 3. Load in Chrome (Developer Mode)

1. Open `chrome://extensions/`
2. Enable **Developer mode**
3. Click **Load unpacked** → select the `extension/` directory

---

## Usage

### Setting Up (Extension Popup)

1. **Create Master Password** — Encrypts your vault locally with AES-256-GCM (PBKDF2, 600k rounds).
2. **Import Compromised Wallet** — Address + private key of the drained account.
3. **Set Recovery Destination** — Your safe cold / hardware wallet address (no key needed).
4. **Configure Gas Sponsor** — A funded wallet that pays rescue transaction fees.

### Executing a Recovery

When you interact with an airdrop claim portal or trigger a sweep, AntiDrain:

1. Intercepts the `eth_sendTransaction` call
2. Simulates the outcome (advisory)
3. Opens a **confirmation modal** showing exactly what will happen:

```text
Network:              Base Mainnet
Compromised wallet:   0x1234...5678
Recovery destination: 0xabcd...ef01
Assets expected:      +1,500 USDC, +0.12 ETH
Sponsor pays:         ~$0.42 (gas)
Victim pays:          $0
Delegation:           Temporary (revoked immediately)
Simulation:           Passed (advisory)
```

4. On confirmation, the sponsor broadcasts the rescue and revokes delegation atomically.

---

## Testing & Verification
 
```bash
# Smart contract tests (122 Foundry tests across 14 suites)
forge test --root contracts

# CLI & security invariant suites (144 tests)
cd cli && node dist/test/deterministic-verification-suite.js && node --test dist/test/multi-chain-compatibility.js && cd ..

# Real Chromium browser adversarial suite (77 tests)
cd cli && node dist/test/real-browser-adversarial-suite.js && cd ..
```

---

## Project Structure

```
AntiDrain/
├── contracts/                # Solidity smart contracts & Foundry test suite
│   ├── src/                  #   UniversalRecoveryDelegate.sol, BatchExecutor.sol
│   ├── test/                 #   122 Foundry tests (14 suites: fuzz, adversarial, matrix)
│   └── script/               #   CREATE2 deployment script
├── extension/                # Chrome Manifest V3 browser companion
│   ├── src/                  #   TypeScript source (background, content, inpage, popup, vault)
│   ├── manifest.json         #   Extension configuration (storage permission only)
│   ├── icons/                #   Extension icons
│   └── scripts/              #   Build & packaging scripts
├── cli/                      # TypeScript CLI coordinator
│   ├── src/                  #   Rescue orchestrator, EIP-712 encoder, bridge, simulator
│   └── src/test/             #   Deterministic verification & adversarial test suites
├── scripts/                  # Build verification & audit scripts
├── LICENSE                   # MIT License
├── PRIVACY.md                # Privacy policy (zero telemetry, local-only)
├── SECURITY.md               # Vulnerability disclosure via GitHub Security Advisories
└── README.md                 # This file
```

---

## Security Model & Disclosures

### What AntiDrain Protects Against
- ✅ Malicious webpage JavaScript accessing private keys
- ✅ Signing oracle attacks (`personal_sign`, `eth_sign`, `eth_signTypedData` → blocked with Error 4100)
- ✅ Frontrunning via public mempool exposure (restricted broadcast transport)
- ✅ Unauthorized asset routing (on-chain dual-authorization: sponsor + EIP-712 intent)
- ✅ Storage collisions in delegated EOAs (ERC-7201 unstructured nonces)
- ✅ Reentrancy during token callbacks (EIP-1153 transient lock)

### What AntiDrain Cannot Protect Against
- ⚠️ **Compromised browser or OS** — A fully compromised browser binary or operating system can read process memory.
- ⚠️ **Compromised extension update** — A malicious Chrome Web Store update could modify the background service worker.
- ⚠️ **Hardware-level attacks** — AntiDrain is a software wallet. Decrypted keys reside in V8 engine memory while unlocked. It is not equivalent to a hardware wallet or secure enclave.

### Broadcast Transport Reality
- **Ethereum**: Flashbots Protect provides true private builder auction with MEV privacy.
- **L2 Rollups** (Base, Arbitrum, Optimism, Mantle): Direct sequencer feeds with centralized FIFO ordering. No public p2p mempool.
- **Polygon PoS**: Dedicated Bor RPC gateway. **No cryptographic MEV privacy**.

Pre-broadcast simulation (`eth_call`) is an **advisory estimation** only. Asset destinations and transaction integrity are enforced on-chain by the smart contract.

### Operational Status & Disclaimers
> **MAINNET LIVE CANARY: NOT PERFORMED / UNVERIFIED**
> 
> AntiDrain has been extensively verified on local Anvil forks with adversarial test vectors, and on public testnets (Base Sepolia). No live mainnet transactions with real capital have been performed or verified. All real rescues must be simulated via `eth_call` and confirmed by a human operator prior to broadcast.

---

## Contributing

Contributions are welcome. Please open an issue to discuss proposed changes before submitting a pull request.

---

## Security & Responsible Disclosure

If you discover a vulnerability, please report it privately via [GitHub Security Advisories](https://github.com/MdRakibHassanJoy/AntiDrain/security/advisories/new).

See [SECURITY.md](SECURITY.md) for details.

---

## License

[MIT License](LICENSE) — Copyright (c) 2026 AntiDrain Contributors
