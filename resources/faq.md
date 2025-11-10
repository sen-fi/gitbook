# Frequently Asked Questions

## General

### What is Sen Protocol?

Sen is a privacy-first, agent-to-agent payment protocol that enables autonomous shopping assistants to discover deals, negotiate with vendors, and execute payments on behalf of users - all while maintaining hardware-level privacy guarantees.

### How is Sen different from other payment platforms?

1. **Privacy-First**: All financial data encrypted in hardware TEEs - even Sen cannot see user wealth
2. **Agent-Driven**: Autonomous agents handle negotiation and payments without constant human intervention
3. **Vendor Blindness**: Vendors never see individual users, only aggregate metrics
4. **Event-Driven**: Agents wake on-demand, no wasted uptime

### Is Sen a blockchain?

No. Sen is a protocol built on top of Solana blockchain, using MagicBlock's Private Ephemeral Rollups (PER) for privacy. Think of it as a privacy-focused L2.

### Is Sen decentralized?

Sen is progressively decentralizing:
- **Current**: Centralized TEE validator (MagicBlock)
- **Near-term**: Multi-operator agent network
- **Long-term**: Fully decentralized matching and settlement

## Privacy & Security

### How private is Sen?

Sen provides **hardware-level privacy** through Trusted Execution Environments (TEEs):

**What's Private**:
- ✅ Transfer amounts
- ✅ Sender/recipient identities
- ✅ Vault balances
- ✅ AI recommendations
- ✅ Browsing behavior

**What's Public**:
- ❌ Deposit/withdrawal to L1
- ❌ Settlement to external wallets
- ❌ Deal postings (intentionally public)

See [Privacy Architecture](../architecture/privacy.md) for details.

### Can Sen see my financial data?

**No**. All sensitive data is encrypted in hardware TEEs. Sen's servers process encrypted blobs and cannot decrypt without your keys (which never leave your device or agent TEE).

### Can vendors track me?

**No**. Vendors never see:
- Your identity
- Your agent ID
- Your browsing history
- Individual purchase decisions

Vendors only see aggregate metrics (e.g., "23 agents matched, 5 purchases").

### What if the TEE is compromised?

TEEs (Intel TDX, AWS Nitro) provide hardware-level isolation with cryptographic attestation. Compromising a TEE would require:
- Breaking hardware security features (extremely difficult)
- Access to physical hardware (protected by cloud providers)
- Defeating attestation (cryptographically verified)

Even if compromised, damage is limited to that TEE instance - your device keys are separate.

### How does Sen comply with regulations?

Sen balances privacy with compliance:
- **KYC/AML**: Identity verification at onboarding (via Privy)
- **Transaction Monitoring**: Suspicious pattern detection
- **Audit Trails**: Available to authorized regulators
- **Privacy ≠ Anonymity**: Users are known to Sen, private from vendors

## Technical

### What blockchain does Sen use?

**Solana** - for its speed, low cost, and TEE integration via MagicBlock.

We also use **MagicBlock PER** (Private Ephemeral Rollups) as a privacy-focused L2.

### What are Private Ephemeral Rollups (PER)?

PER is MagicBlock's L2 solution using Intel TDX TEEs:
- Private state updates in hardware enclaves
- Encrypted transfers invisible to blockchain
- Periodic L1 settlement for security

See [PER Architecture](../architecture/privacy/per.md) for details.

### Can I use Sen with other wallets?

Yes! Sen supports:
- **PER deposits** (recommended for privacy)
- **External wallets** (MetaMask, Phantom, etc.)
- **Hybrid flows** (PER for Sen→Sen, L1 for external)

### What tokens does Sen support?

Currently:
- **USDC** (Solana devnet: `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU`)
- **SOL** (coming soon)
- **Other SPL tokens** (planned)

### How fast are payments?

| Type | Speed | Privacy |
|------|-------|---------|
| PER transfer | ~2s | High (TEE) |
| L1 transfer | ~5s | Low (public) |
| Multi-vendor comparison | ~5s | High |

## Agents

### What can agents do?

Sen agents can:
- **Browse deals** anonymously
- **Compare prices** across vendors
- **Negotiate** better prices autonomously
- **Execute payments** via PER
- **Learn preferences** over time

See [Agent Infrastructure](../architecture/agents.md) for details.

### Do agents have access to my money?

Agents operate within **spending limits** you set:
- Daily/weekly/monthly caps
- Per-transaction limits
- Category-specific budgets
- Approval required for large purchases

### Can I disable agents?

Yes. Agents are fully optional:
- Disable entirely (manual mode)
- Require approval for all purchases
- Set spending limits to $0
- Revoke permissions anytime

### How do agents learn?

**Current**: Session memory only (in-memory, cleared on app close)

**Future**: Encrypted persistent memory in TEE for long-term preference learning

Agents NEVER share your data with vendors or third parties.

## Bazaar Marketplace

### How does deal discovery work?

1. **Browse anonymously** - No auth, no tracking
2. **Evaluate locally** - Check eligibility in your agent
3. **Initiate purchase** - Vendor learns about you only when you buy

See [Deal Discovery](../marketplace/discovery.md) for details.

### What are "ephemeral signals"?

Privacy-preserving data for personalized deals:
- **Ephemeral IDs** rotate every 1-6 hours (no tracking)
- **Coarse location** (district/city, not GPS)
- **LTV tiers** (high/medium/low, not exact $)
- **Opt-in only** (you control sharing)

See [Signal System](../marketplace/matching/signals.md) for details.

### Can I opt out of personalized deals?

Yes. You can:
- Disable signal sharing (browse only)
- Configure location granularity (city vs district vs ward)
- Choose which categories to share LTV for
- Delete signals anytime

### How do vendors benefit?

Vendors get:
- **Targeted deals** (reach high-value customers)
- **Aggregate metrics** (conversion rates, redemptions)
- **No platform fees** (in beta)
- **Privacy compliance** (built-in)

## X.402 Protocol

### What is X.402?

X.402 is Sen's agent-to-agent protocol for autonomous negotiation and payment, inspired by HTTP 402 Payment Required.

See [X.402 Overview](../protocol/overview.md) for details.

### Can I integrate my own agents?

Yes! X.402 is an open protocol. You can:
- Build custom user agents
- Integrate vendor agents
- Use Sen SDK for easy integration

See [Agent API](../api/agent-api.md) for documentation.

### Is X.402 compatible with other protocols?

X.402 is designed to be interoperable:
- HTTP-based (RESTful)
- JSON payloads
- Standard authentication (OAuth, JWT)
- Extensible message types

## Development

### Is Sen open source?

**Yes**. All code is MIT licensed:
- Core protocol
- SDK
- Documentation
- Example integrations

Closed during beta for security auditing, will open source at mainnet launch.

### How do I integrate Sen?

1. Install SDK: `npm install @sen-protocol/sdk`
2. Initialize agent: `new X402Negotiator({ ... })`
3. Browse deals: `discovery.browseDeals({ ... })`
4. Execute payments: `negotiator.initiatePurchase({ ... })`

See [Quick Start](../README.md#quick-start) for code examples.

### Where can I test?

- **Devnet**: Solana devnet + MagicBlock PER testnet
- **Testnet**: Coming soon
- **Mainnet**: Q1 2025

### How do I report bugs?

- **GitHub Issues**: [github.com/sen-fi/sen-protocol/issues](https://github.com/sen-fi/sen-protocol/issues)
- **Discord**: Join our community
- **Email**: security@sen.fi (for security issues)

## Roadmap

### What's next for Sen?

**Q4 2024**:
- ✅ Privacy-first deal discovery
- ✅ Anonymous browsing
- ✅ X.402 protocol v1
- ⏳ zkSNARK proofs for LTV

**Q1 2025**:
- Mainnet launch
- Multi-chain support (Ethereum, Polygon)
- Advanced AI negotiation
- Decentralized matching

**Q2-Q3 2025**:
- Cross-chain privacy bridges
- Fully homomorphic encryption (FHE)
- Decentralized agent network
- Global marketplace

See [Roadmap](../protocol/overview.md#roadmap) for details.

### Will there be a token?

Not currently planned. Sen focuses on utility, not speculation.

If a token is introduced, it would serve a clear purpose:
- Agent collateral/staking
- Governance
- Economic security (slashing)

---

## Still have questions?

- **Discord**: Join our community
- **Email**: support@sen.fi
- **GitHub Discussions**: [github.com/sen-fi/sen-protocol/discussions](https://github.com/sen-fi/sen-protocol/discussions)

[Back to Resources](../resources/)
