# Agent API & MCP Guide

## MCP Server

Connect at: `http://localhost:3001/mcp` (or your deployed backend URL)

```json
{
  "mcpServers": {
    "token-minter": {
      "url": "http://localhost:3001/mcp"
    }
  }
}
```

### MCP Tools

| Tool | Description |
|------|-------------|
| `get_supported_chains` | List supported blockchains and their chain IDs |
| `pin_token_metadata` | Upload ERC-7572 metadata to IPFS, returns `ipfs://` URI |
| `build_create_token_transaction` | Generate transaction calldata for token deployment |
| `list_tokens` | Query all tokens via the Subgraph |
| `validate_promo_code` | Validate a promo code and get discount signature |

### Typical MCP Flow (Celo Native)

```
1. validate_promo_code (optional)
2. pin_token_metadata → ipfs://Qm...
3. build_create_token_transaction → { to, data, value }
4. User signs and broadcasts transaction
```

---

## REST API

Base URL: `http://localhost:3001` (or your deployed backend URL)

### Pin Metadata

```
POST /api/metadata/pin
Content-Type: application/json

{
  "name": "Token Name",
  "symbol": "TKN",
  "decimals": 18,
  "description": "Optional description",
  "website": "https://example.com",
  "image": "ipfs://... or https://..."
}

Response:
{
  "data": {
    "metadataURI": "ipfs://QmXxx..."
  }
}
```

### Get Calldata — Celo Native

```
POST /api/tokens/42220/create/calldata
Content-Type: application/json

{
  "owner": "0xYourWalletAddress",
  "name": "Community Token",
  "symbol": "COMM",
  "decimals": 18,
  "initialSupply": "1000000",
  "maxSupply": "0",
  "metadataURI": "ipfs://Qm..."
}

Response:
{
  "data": {
    "to": "0xFactoryAddress",
    "data": "0x...",
    "value": "1000000000000000"
  }
}
```

### Get Calldata — Full Deployment (Ethereum Enabled)

```
POST /api/tokens/full-deployment/calldata
Content-Type: application/json

{
  "tokenType": "ethereum-enabled",
  "owner": "0x...",
  "name": "My Token",
  "symbol": "MTK",
  "decimals": 18,
  "initialSupply": "1000000",
  "maxSupply": "0",
  "metadataURI": "ipfs://Qm..."
}
```

### Validate Promo Code

```
POST /api/promo/validate
Content-Type: application/json

{
  "code": "LAUNCH2024",
  "userAddress": "0x...",
  "chainId": 42220,
  "factoryAddress": "0x..."
}

Response includes: promoFee, promoNonce, expiresAt, signature
```

### List Tokens

```
GET /api/tokens?chainId=42220&owner=0x...
```

---

## Contract: createToken (Celo L2)

```solidity
function createToken(
    address owner_,
    string memory name_,
    string memory symbol_,
    uint8 decimals_,
    uint256 initialSupply_,
    uint256 maxSupply_,
    string memory metadataURI_
) external payable returns (address tokenAddress);
```

## Contract: createTokenWithPromo

```solidity
function createTokenWithPromo(
    address owner_,
    string memory name_,
    string memory symbol_,
    uint8 decimals_,
    uint256 initialSupply_,
    uint256 maxSupply_,
    string memory metadataURI_,
    uint256 promoFee_,
    bytes32 promoNonce_,
    uint256 expiresAt_,
    bytes memory signature_
) external payable returns (address tokenAddress);
```

## Listening for Token Creation

```typescript
import { decodeEventLog } from 'viem';

const receipt = await publicClient.waitForTransactionReceipt({ hash });

for (const log of receipt.logs) {
  try {
    const event = decodeEventLog({
      abi: L2SuperChainTokenFactoryABI,
      data: log.data,
      topics: log.topics,
    });
    if (event.eventName === 'TokenCreated') {
      console.log('Token address:', event.args.tokenAddress);
    }
  } catch {}
}
```

## Common Error Codes

| Error | Cause | Fix |
|-------|-------|-----|
| `InsufficientFee` | Value sent is less than creation fee | Use the `value` field from calldata response |
| `InvalidOwner` | owner_ is zero address | Provide a valid wallet address |
| `InvalidSupply` | initialSupply > maxSupply | Fix supply values, or set maxSupply to 0 |
| `InvalidSignature` | Bad promo signature | Re-run validate_promo_code |
| `PromoExpired` | Promo code expired | Re-validate or use a different code |

## Gas Recommendations

- Default gas limit: `600000`
- If gas estimation fails, override with `600000` and retry
