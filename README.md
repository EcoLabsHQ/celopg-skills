# Token Manager — Claude Skill

> **Deploy and manage ERC-20 tokens on Celo and Ethereum directly through Claude AI.**

This skill teaches Claude how to use the [Token Manager](https://token.celopg.eco) — guiding users through token creation, cross-chain deployment, and L2-to-L1 migration using natural conversation.

---

## What This Skill Does

Once installed, you can talk to Claude like this:

> *"Create a community token called COMM on Celo with 1 million supply"*
> *"Deploy a cross-chain token on Ethereum and Celo with bridge support"*
> *"Migrate my existing Celo token to Ethereum"*
> *"Apply my promo code and create a token"*

Claude will walk you through every step — wallet setup, parameter selection, transaction signing, and troubleshooting — using the Token Manager's UI, REST API, or MCP server.

---

## Supported Flows

| Flow | Description | Time | Fee |
|------|-------------|------|-----|
| **Celo Native** | Token on Celo L2 only | ~5 sec | ~0.001 CELO |
| **Ethereum Enabled** | Token on Ethereum + Celo with bridge | ~2 min + 20 min bridge | ~0.01 ETH |
| **L2 → L1 Migration** | Upgrade existing Celo token to have an Ethereum counterpart | ~2 min + bridge time | ~0.01 ETH |

---

## Installation

### Option A: Upload to Claude.ai (Recommended for most users)

1. **Download** `token-manager.zip` from the latest [Release](../../releases)
2. **Upload to Claude.ai**:
   - Go to [claude.ai](https://claude.ai) → click your profile → **Settings**
   - Navigate to **Capabilities → Skills**
   - Click **Upload skill** and select `token-manager.zip`
3. **Enable the skill** — toggle it ON in your skills list
4. **Test it** — ask Claude: *"Help me create a token on Celo"*

### Option B: Claude Code (for developers)

```bash
# Clone the repo
git clone https://github.com/EcoLabsHQ/celopg-skills.git

# Copy the skill to your Claude Code skills directory
cp -r celopg-skills/token-manager ~/.claude/skills/
```

### Option C: MCP + Skill (Full AI Agent Setup)

For the best experience, use this skill together with the Token Manager MCP server. The MCP server gives Claude live access to your Token Manager; the skill gives Claude the knowledge to use it correctly.

**Step 1 — Add the MCP server** to your Claude config:

```json
{
  "mcpServers": {
    "token-minter": {
      "url": "https://token.celopg.eco/mcp"
    }
  }
}
```

**Step 2 — Install the skill** using Option A or B above.

**Step 3 — Ask Claude to create a token** — it will use both the skill (for workflow knowledge) and the MCP server (for live API calls) together.

---

## What's in the Skill Folder

```
token-manager/
├── SKILL.md                          # Main skill — all workflows and instructions
├── references/
│   ├── api-guide.md                  # MCP tools + REST API reference
│   └── contract-addresses.md         # Factory and bridge contract addresses
└── assets/
    └── token-planning-template.md    # Pre-launch checklist for token parameters
```

---

## Why Use the Skill + MCP Together?

| Without Skill | With Skill + MCP |
|---------------|-----------------|
| You explain your workflow each conversation | Claude knows the workflow automatically |
| Manual API calls and parameter lookups | Claude calls the API and formats parameters |
| Generic responses | Celo-specific guidance with exact addresses and fees |
| Troubleshooting by trial and error | Claude diagnoses errors with known fixes |

---

## Example Conversations

**Creating a Celo Native token:**
```
You: Create a token called "Farmers Market Token" with symbol FMT,
     1 million supply, on Celo

Claude: I'll walk you through creating FMT on Celo using the Token
        Manager. Let's start by gathering your token details...
        [guides through each step]
```

**Applying a promo code:**
```
You: I have promo code CELOPG2024 — can you create my token for free?

Claude: I'll validate that promo code first, then build your token
        transaction with the discounted fee applied...
```

**Troubleshooting:**
```
You: I got an "InsufficientFee" error when deploying

Claude: This means the transaction value sent was less than the creation fee.
        Check the `value` field returned by build_create_token_transaction
        and make sure you're sending that exact amount with the transaction.
```

---

## Requirements

- A [Claude.ai](https://claude.ai) account (Free, Pro, or Team)
- A Celo-compatible wallet (MetaMask, WalletConnect, or Valora)
- CELO for gas fees (~0.01 CELO is plenty for a Celo Native token)
- ETH if using the Ethereum Enabled flow (~0.01 ETH)

---

## Links

- **Token Manager App**: [token.celopg.eco](https://token.celopg.eco)
- **Celo Documentation**: [docs.celo.org](https://docs.celo.org)
- **CeloScan Explorer**: [celoscan.io](https://celoscan.io)
- **Celo Public Goods**: [celopg.eco](https://celopg.eco)

---

## Contributing

Found a bug or want to add a workflow? Open an issue or pull request. Skill improvements are especially welcome for:
- New error messages and fixes
- Additional MCP tool examples
- Testnet / Alfajores instructions

---

## License

MIT
