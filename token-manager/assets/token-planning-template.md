# Token Planning Template
Use this before launching to make sure you've thought through every parameter.

---

## Basic Parameters

| Parameter | Your Value | Notes |
|-----------|-----------|-------|
| Token Name | | Full name, e.g. "Community Token" |
| Symbol | | 3-5 chars, e.g. "COMM" |
| Decimals | 18 | Standard is 18; use 6 for stablecoin-like tokens |
| Initial Supply | | Tokens minted at deployment |
| Max Supply | | 0 = unlimited; otherwise sets hard cap |

## Deployment Target

- [ ] **Celo Native** — Celo only, fast and cheap (~0.001 CELO)
- [ ] **Ethereum Enabled** — ETH + Celo with bridge (~0.01 ETH)
- [ ] **L2 Migration** — Upgrade existing Celo token to have L1 counterpart

## Metadata (Optional but Recommended)

| Field | Your Value |
|-------|-----------|
| Description | |
| Website | |
| Logo image | (upload file or IPFS/URL) |

## Supply Strategy Checklist

- [ ] Who receives the initial supply? (your wallet = owner address)
- [ ] Is the max supply appropriate for your tokenomics?
- [ ] If Ethereum Enabled: Do you want to bridge initial supply to L2 immediately?

## Pre-Deployment Wallet Checklist

**For Celo Native:**
- [ ] Wallet connected to Celo Mainnet (Chain ID: 42220)
- [ ] At least 0.01 CELO in wallet for gas

**For Ethereum Enabled:**
- [ ] Wallet connected to Ethereum Mainnet
- [ ] At least 0.01 ETH in wallet (Ethereum step)
- [ ] At least 0.01 CELO in wallet (Celo step)
- [ ] Both Ethereum and Celo networks configured in wallet

## Post-Deployment Checklist

- [ ] Save token contract address
- [ ] Verify on CeloScan: https://celoscan.io/token/YOUR_ADDRESS
- [ ] Add token to your wallet (custom token, paste contract address)
- [ ] Share contract address with your community
- [ ] (If Ethereum Enabled) Note both L1 and L2 addresses

## Promo Code

- [ ] Do you have a promo code? ___________
- [ ] Validate it before deployment at: POST /api/promo/validate
