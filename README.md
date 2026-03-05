# YAMAGI DeFi — OPNet BTC L1

React + Vite frontend for PILL/BTC swaps on OPNet Testnet.

## Stack

| Package | Role |
|---|---|
| `opnet` | JSONRpcProvider, getContract, ABIs |
| `@btc-vision/bitcoin` | Bitcoin library (OPNet fork) |
| `@btc-vision/transaction` | Transaction types & ABI data types |
| `@btc-vision/walletconnect` | Wallet connection modal |
| `react` + `vite` | UI framework + build tool |

## Quick Start

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173)

## Project Structure

```
src/
  components/
    Navbar.jsx        # Navigation + wallet button
    Ticker.jsx        # Live price ticker bar
    StatsBar.jsx      # TVL / volume / price stats
    HeroPanel.jsx     # Landing page
    SwapPanel.jsx     # BTC ↔ PILL swap UI
    PoolsPanel.jsx    # Liquidity pools + staking
    Footer.jsx        # Footer
  hooks/
    useWallet.js      # @btc-vision/walletconnect integration
    useSwap.js        # Swap state + OPNet simulate/send pattern
  lib/
    opnet.js          # JSONRpcProvider singleton + simulateAndSend helper
  styles/
    globals.css       # Theme vars + MANDATORY WalletConnect CSS fix
```

## OPNet Contract Integration

To connect a real contract, replace the mock in `useSwap.js`:

```js
import { getContract } from 'opnet'
import { getProvider } from '../lib/opnet'

const provider = getProvider() // READ-ONLY — never use WalletConnect for reads
const contract = getContract(PILL_CONTRACT_ADDR, PILL_ABI, provider, 'testnet', address)

// Always simulate first
const sim = await contract.swap(amountIn, amountOutMin)
if ('error' in sim) { /* handle revert */ return }

// Send with null signers — frontend never holds private keys
const tx = await sim.sendTransaction({
  signer: null,
  mldsaSigner: null,
  refundTo: address,
})
```

## Networks

- **Testnet RPC**: `https://testnet.opnet.org`
- **Mainnet RPC**: `https://mainnet.opnet.org`

## WalletConnect CSS Fix

The mandatory popup fix is in `src/styles/globals.css`.
**Do not remove it** — without it the WalletConnect modal renders invisibly.

## Notes

- Never use `bitcoinjs-lib`, `ethers`, `web3`, `@metamask/sdk`, or `window.ethereum`
- Never use the WalletConnect provider for reads — only for signing
- Never store private keys in frontend code
