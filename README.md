# Sen Protocol Documentation

Welcome to the **Sen Protocol** documentation - a global mobile financial super app that bridges traditional payment systems with Web3 technology through privacy-first architecture and AI-powered autonomous agents.

## What is Sen?

Sen is an agentic, off-chain negotiated, on-chain settled financial platform designed to build wealth for everyone. It combines:

- **Privacy-First Architecture**: Hardware-level encryption using TEEs (Trusted Execution Environments)
- **Agent-to-Agent Protocol**: Autonomous negotiation via X.402 protocol
- **Event-Driven Design**: Agents wake on-demand for efficiency
- **Hybrid AI**: Sensitive operations in TEE, general tasks via external APIs

## Core Features

### 🔒 Privacy Infrastructure
Sen provides **zero-knowledge privacy** through:
- MagicBlock PER (Private Ephemeral Rollups) on Solana
- Intel TDX TEE for encrypted compute
- Hardware-level guarantees - even Sen cannot see user wealth

### 🤖 Autonomous Agents
Event-driven agents that:
- Research products and compare deals
- Negotiate with vendors autonomously
- Execute payments privately
- Learn user preferences over time

### 🏪 Privacy-Preserving Marketplace
Bazaar/Agora marketplace where:
- Vendors post deals publicly
- Users browse anonymously
- Matching happens in TEE
- Vendors never see individual users

## Documentation Structure

This documentation is organized into the following sections:

### Architecture
- **[Privacy Architecture](architecture/privacy.md)** - TEE, PER, and encryption
- **[Agent Infrastructure](architecture/agents.md)** - Event-driven agent design
- **[Payment System](architecture/payments.md)** - Private payment flows

### Protocol
- **[X.402 Overview](protocol/overview.md)** - Agent-to-agent negotiation protocol
- **[Message Types](protocol/messages.md)** - Quote requests, offers, counter-offers
- **[Payment Flow](protocol/payment-flow.md)** - End-to-end payment execution

### Marketplace
- **[Bazaar Architecture](marketplace/bazaar.md)** - Privacy-preserving deal platform
- **[Deal Discovery](marketplace/discovery.md)** - Anonymous browsing
- **[Matching Engine](marketplace/matching.md)** - TEE-based matching

### API Reference
- **[Client SDK](api/client-sdk.md)** - JavaScript/TypeScript SDK
- **[Agent API](api/agent-api.md)** - Agent integration API
- **[Vendor API](api/vendor-api.md)** - Vendor integration

## Quick Start

```typescript
import { X402Negotiator, DealDiscovery } from '@sen-protocol/sdk';

// Initialize agent
const agent = new X402Negotiator({
  userId: 'user123',
  agentId: 'agent456',
  config: { autoNegotiate: true }
});

// Discover deals anonymously
const discovery = new DealDiscovery();
const deals = await discovery.browseDeals({ category: 'coffee' });

// Initiate purchase (privacy-preserving)
const confirmation = await agent.initiatePurchase({
  deal: deals[0],
  paymentMethod: 'per',
  includeProofs: true
});
```

## Use Cases

### P2P Payments
Private transfers between Sen users using PER - amounts, sender, and recipient all hidden in TEE.

### AI Shopping Assistant
Autonomous agents research products, negotiate deals, and make purchases on your behalf.

### Privacy-First Commerce
Shop at local vendors without revealing your identity or purchase history.

## Security & Privacy

Sen is built on three core principles:

1. **Zero-Knowledge Privacy**: All sensitive compute happens in hardware TEEs
2. **Vendor Blindness**: Vendors never see individual users or agents
3. **Hardware-Level Security**: Intel TDX and AWS Nitro provide cryptographic guarantees

## Get Involved

- **GitHub**: [github.com/sen-fi](https://github.com/sen-fi)
- **Discord**: Join our community
- **Twitter**: [@sen_protocol](https://twitter.com/sen_protocol)

## License

MIT License - see [LICENSE](LICENSE) for details.
