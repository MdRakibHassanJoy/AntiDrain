# AntiDrain — Universal EVM Chain Support & Capability Matrix

## 1. Multi-Chain EVM Architecture

AntiDrain decouples **EVM Network Recognition** from **Production Rescue Capability**:

* Any standard EVM network can be recognized by the wallet for basic balance and transaction observation.
* EIP-7702 Rescue is strictly enabled on networks that have completed full verification of the deterministic CREATE2 delegate contract (`0x60BAf255624BEE5629e5D86Ae8976aF19795A314`).

---

## 2. Universal EVM Capability Matrix

| Network | Chain ID | Gas Token | EIP-7702 Status | Delegate Address | Bytecode Hash Verified | Rescue Capability |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Ethereum Mainnet** | `1` | **ETH** | ✅ (Pectra) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Base Mainnet** | `8453` | **ETH** | ✅ (May 2025) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Base Sepolia (Testnet)** | `84532` | **ETH** | ✅ (May 2025) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Arbitrum One** | `42161` | **ETH** | ✅ (ArbOS 40) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Optimism** | `10` | **ETH** | ✅ (Isthmus) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Polygon PoS** | `137` | **POL** | ✅ (PIP-61) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Mantle** | `5000` | **MNT** | ✅ (Everest) | `0x60BAf2...A314` | `0x4933ce...a2dd` | 🟢 **ENABLED** |
| **Unknown EVM** | *Any* | *Any* | ❓ | *None* | *None* | 🔴 **DISABLED (FAIL-CLOSED)** |

---

## 3. Broadcast Transport Details

* **Ethereum (Chain 1):** Routes via Flashbots Protect for MEV privacy.
* **L2 Rollups (Base, Arbitrum, Optimism, Mantle):** Routes via official sequencer endpoints with centralized FIFO execution.
* **Polygon PoS (Chain 137):** Dedicated Bor RPC gateway (**POL** native gas token; no cryptographic MEV privacy).
