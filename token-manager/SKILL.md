---
name: token-manager
description: Deploys and manages ERC-20 tokens on Celo and Ethereum using the High Velocity Token Manager (token.celopg.eco). Use when user asks to "create a token", "deploy a token on Celo", "launch a token", "create a cross-chain token", "bridge a token to Ethereum", "migrate my Celo token to L1", "create an Ethereum enabled token", "make a community token", "set up token with bridge support", "apply a promo code to token creation", or mentions "HVTM", "token manager", "Celo native token", "L2 token", "superchain token", or "celopg token". Handles three flows: Celo Native (Celo only), Ethereum Enabled (ETH + Celo with bridge), and L2-to-L1 Migration. Also supports MCP server and REST API for programmatic token creation.
metadata:
  author: Celo Public Goods
  version: 1.0.0
  documentation: https://token.celopg.eco
  mcp-server: token-minter
---

# High Velocity Token Manager

Deploy and manage ERC-20 tokens on Celo and Ethereum at [token.celopg.eco](https://token.celopg.eco).

---

## Choose Your Flow

| I want to... | Use this flow |
|---|---|
| Create a token only on Celo (fast, cheap) | **Flow 1: Celo Native** |
| Create a token on both Ethereum + Celo with bridging | **Flow 2: Ethereum Enabled** |
| Upgrade my existing Celo token to work on Ethereum | **Flow 3: L2 to L1 Migration** |

---

## Flow 1: Celo Native Token

**Best for:** Community tokens, local currencies, cost-effective deployment.
**Time:** ~5 seconds | **Fee:** ~0.001 CELO

### Before you start, gather:
- Token **name** (e.g. "Community Token")
- Token **symbol** (e.g. "COMM", 3-5 chars)
- **Decimals** (default: 18)
- **Initial supply** (tokens minted at launch, e.g. "1000000")
- **Max supply** (0 = unlimited)
- **Logo image** (optional)
- **Description** and **website URL** (optional metadata)
- A Celo-compatible wallet with at least 0.01 CELO for gas

### Steps

**Step 1 — Connect wallet**
1. Go to [token.celopg.eco](https://token.celopg.eco)
2. Click **Connect Wallet** → select MetaMask, WalletConnect, or Valora
3. Ensure you are on **Celo Mainnet** (Chain ID: 42220)

**Step 2 — Select "Celo Native"**

**Step 3 — Fill in token details**
- Name, symbol, decimals, initial supply, max supply
- Optionally upload a logo and add description/website

**Step 4 — Review**
- Double-check all parameters — the token address is deterministic based on name/symbol/creator and cannot be changed after deploy

**Step 5 — Deploy**
- Click **Deploy** and sign the transaction in your wallet
- Wait ~5 seconds for block confirmation
- Your token address is shown on success

**What happens under the hood:**
1. Logo uploaded to CDN (if provided)
2. Metadata pinned to IPFS (ERC-7572 compliant)
3. `L2SuperChainTokenFactory.createToken()` called on Celo (Chain ID 42220)
4. Initial supply minted to your wallet
5. Logo copied from temp hash to token address

### Via MCP
```
User: "Create a token called Community Token with symbol COMM on Celo with 1 million supply"

Agent:
1. pin_token_metadata → gets ipfs://Qm...
2. build_create_token_transaction → gets transaction calldata
3. Returns transaction for user to sign
```

### Via REST API
```bash
# Step 1: Pin metadata
POST /api/metadata/pin
{
  "name": "Community Token",
  "symbol": "COMM",
  "decimals": 18,
  "description": "A community governance token"
}
# Returns: { "data": { "metadataURI": "ipfs://Qm..." } }

# Step 2: Get transaction calldata
POST /api/tokens/42220/create/calldata
{
  "owner": "0xYourAddress",
  "name": "Community Token",
  "symbol": "COMM",
  "decimals": 18,
  "initialSupply": "1000000",
  "maxSupply": "0",
  "metadataURI": "ipfs://Qm..."
}
# Returns: { "data": { "to": "0x...", "data": "0x...", "value": "..." } }

# Step 3: Sign and broadcast the transaction with your wallet
```

---

## Flow 2: Ethereum Enabled Token (Cross-Chain)

**Best for:** Tokens needing Ethereum DeFi access, cross-chain projects.
**Time:** ~2 min setup + 15-20 min bridge finalization | **Fee:** ~0.01 ETH

### Before you start, gather:
- Same token info as Flow 1
- A wallet with **ETH on Ethereum mainnet** (~0.01 ETH) AND **CELO** for L2 step
- Decision: bridge initial supply to Celo L2 immediately, or later?

### Steps

**Steps 1-3:** Same as Flow 1 (connect wallet, select "Ethereum Enabled", fill details)

**Step 4 — Deploy on Ethereum L1**
- Sign transaction on **Ethereum** network
- `L1TokenFactory.createToken()` is called
- Initial supply minted to your wallet on Ethereum
- Fee: ~0.01 ETH

**Step 5 — Deploy bridged token on Celo L2**
- Wallet auto-switches to **Celo** network
- `L2SuperChainTokenFactory.createTokenWithBridge()` is called
- This step is FREE (cost already covered by L1 payment)
- Links L2 token to L1 via L2 Bridge: `0x4200000000000000000000000000000000000010`

**Step 6 — Bridge initial supply to L2 (optional)**
- Sign **Approve** transaction on Ethereum (allows bridge to move tokens)
- Sign **Bridge** transaction on Ethereum (`L1StandardBridge.bridgeERC20To()`)
- Wait 15-20 minutes for bridge finalization on Celo

**Result:**
- L1 address: `0x...` (Ethereum, holds initial supply)
- L2 address: `0x...` (Celo, receives bridged tokens)

---

## Flow 3: Migrate Existing L2 Token to L1

**Best for:** Projects that started on Celo and now want Ethereum access.
**Time:** ~2 min + bridge time | **Fee:** ~0.01 ETH
**Warning:** Bridge settings are **irreversible** once set.

### Requirements
- You must be the **owner** of the existing Celo L2 token
- ETH on Ethereum for deployment (~0.01 ETH)
- CELO for two configuration transactions on L2

### Steps

**Step 1 — Select your existing token**
- Connect wallet and select the L2 token you want to migrate

**Step 2 — Create L1 counterpart on Ethereum**
- `L1TokenFactory.createToken()` called with `initialSupply: 0`
- Supply stays on L2; L1 starts with 0 tokens
- Note the new L1 token address

**Step 3 — Configure bridge on L2 (owner only)**
- `L2SuperChainToken.setBridge(0x4200000000000000000000000000000000000010)` on Celo

**Step 4 — Link L1 token on L2 (owner only)**
- `L2SuperChainToken.setRemoteToken(L1_TOKEN_ADDRESS)` on Celo

**Step 5 — Bridge tokens (optional)**
- Use `L2StandardBridge.bridgeERC20To()` to move tokens from L2 to L1

**Result:** Existing Celo token now has an Ethereum counterpart with full two-way bridging.

---

## Promo Codes

All flows support promo codes to reduce or eliminate creation fees.

```bash
POST /api/promo/validate
{
  "code": "YOUR_CODE",
  "userAddress": "0x...",
  "chainId": 42220,
  "factoryAddress": "0x..."
}
# Returns signature data used with createTokenWithPromo()
```

Via MCP: use the `validate_promo_code` tool — it handles the full validation automatically.

---

## MCP Server Setup

Connect Claude or any AI agent to the Token Manager MCP server:

```json
{
  "mcpServers": {
    "token-minter": {
      "url": "http://localhost:3001/mcp"
    }
  }
}
```

**Available MCP tools:**
- `get_supported_chains` — list supported blockchains
- `pin_token_metadata` — upload metadata to IPFS
- `build_create_token_transaction` — generate transaction calldata
- `list_tokens` — query created tokens
- `validate_promo_code` — apply promotional discounts

---

## Troubleshooting

**Wallet won't connect**
- Confirm Celo Mainnet is added to your wallet: Chain ID `42220`, RPC `https://forno.celo.org`
- For cross-chain flows, ensure both Ethereum Mainnet and Celo are configured

**`InsufficientFee` error**
- Celo Native: send at least 0.001 CELO with the transaction
- Ethereum Enabled: send at least 0.01 ETH
- Get the exact fee from the `value` field returned by `build_create_token_transaction`

**`InvalidSupply` error**
- `initialSupply` must be less than or equal to `maxSupply`
- Set `maxSupply` to `0` for unlimited supply

**`InvalidSignature` or `PromoExpired` errors**
- Re-run `validate_promo_code` — promo signatures expire quickly
- Try a different promo code if the current one is expired

**Transaction reverted / gas estimation fails**
- Set gas limit manually to `600000`
- Verify all parameters: owner address is not zero, supply is greater than 0

**Chain switch fails mid-flow (Flow 2)**
- Manually switch network in your wallet and click retry
- Do not close the browser tab during a multi-chain flow

**Bridge tokens not arriving after 20+ minutes**
- Check bridge transaction status on [celoscan.io](https://celoscan.io)
- Verify the bridge transaction was mined on the source chain

**Token not visible in wallet after deploy**
- Add the contract address manually as a custom ERC-20 token
- Verify deployment on [celoscan.io](https://celoscan.io)

---

## Key Addresses & References

| Item | Value |
|---|---|
| Celo Chain ID | `42220` |
| Ethereum Chain ID | `1` |
| L2 Standard Bridge (Celo) | `0x4200000000000000000000000000000000000010` |
| Superchain Token Bridge | `0x4200000000000000000000000000000000000028` |

See `references/contract-addresses.md` for factory deployment addresses per network.
See `references/api-guide.md` for full REST API and MCP documentation.
