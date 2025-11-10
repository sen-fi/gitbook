# Privacy Architecture

Sen's privacy architecture is built on three foundational pillars that work together to provide hardware-level privacy guarantees:

1. **Trusted Execution Environments (TEEs)** - Hardware-isolated secure enclaves
2. **Private Ephemeral Rollups (PER)** - TEE-based L2 for private Solana transactions
3. **End-to-End Encryption** - Multi-layer encryption with cryptographic proofs

## Core Privacy Principle

**Zero-Knowledge by Design**: Sen operates on the principle that **no one** - not even Sen itself - should be able to see user financial data without explicit user consent.

This is achieved through:
- Hardware-level isolation (Intel TDX, AWS Nitro)
- Encrypted compute in TEEs
- Private state channels on PER
- Client-side encryption with user-controlled keys

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (Mobile App)                       │
│  - Public keys only                                          │
│  - Private keys never leave device                           │
│  - Encrypted communication                                   │
└──────────────────┬──────────────────────────────────────────┘
                   │ TLS + E2EE
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              User Agent (TEE Instance)                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Trusted Execution Environment (Intel TDX/Nitro)    │   │
│  │  - Stores sensitive keys                            │   │
│  │  - Decrypts payment data                            │   │
│  │  - Signs transactions                               │   │
│  │  - Encrypted memory                                 │   │
│  │  - Remote attestation                               │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────┬──────────────────────────────────────────┘
                   │ Private channels
                   ▼
┌─────────────────────────────────────────────────────────────┐
│         MagicBlock PER (Private Ephemeral Rollup)            │
│  - Private state in TEE                                      │
│  - Encrypted balances                                        │
│  - Hidden transfer amounts                                   │
│  - Periodic L1 settlement                                    │
└─────────────────────────────────────────────────────────────┘
```

## Privacy Guarantees

### What's Private

✅ **Transfer amounts** - All payment amounts encrypted in TEE
✅ **Sender identity** - User identities hidden from vendors
✅ **Recipient identity** - Recipients only known to participants
✅ **Transaction timing** - No public blockchain traces
✅ **Balance information** - Vault balances encrypted
✅ **AI recommendations** - Generated entirely in TEE
✅ **Browsing behavior** - Anonymous deal discovery
✅ **Negotiation history** - Counter-offers stay private

### What's Public

❌ **Deposit/withdrawal events** - On-chain L1 transactions
❌ **Settlement to external wallets** - Final L1 settlement visible
❌ **Deal postings** - Vendors post deals publicly (intentional)
❌ **Aggregate metrics** - Vendors see total redemptions (privacy-preserving)

### Privacy by Layer

| Layer | Technology | Privacy Guarantee |
|-------|-----------|------------------|
| **Application** | Client-side encryption | User controls keys |
| **Agent** | TEE (Intel TDX/AWS Nitro) | Hardware isolation |
| **Payment** | MagicBlock PER | Encrypted L2 state |
| **Settlement** | Solana L1 | Public (batched) |

## Threat Model

### Protected Against

✅ **Chain analysis** - Payments stay off-chain in PER
✅ **Vendor surveillance** - Vendors never see individual users
✅ **Database compromise** - All stored data encrypted
✅ **Network eavesdropping** - TLS + E2EE throughout
✅ **Sen platform abuse** - Sen cannot decrypt user data
✅ **Malicious agents** - TEE attestation prevents tampering

### Not Protected Against

⚠️ **Device compromise** - If user's device is hacked, local keys at risk
⚠️ **TEE vulnerabilities** - Depends on Intel TDX/AWS Nitro security
⚠️ **User operational security** - User must protect backup phrases
⚠️ **Timing analysis** - Very sophisticated attackers might correlate events

### Trust Assumptions

Sen minimizes trust requirements:

1. **Intel TDX / AWS Nitro** - Must trust hardware TEE implementation
2. **MagicBlock PER** - Must trust TEE validator (currently centralized)
3. **Open source code** - All code auditable, no hidden backdoors
4. **Cryptographic primitives** - Standard algorithms (AES-GCM, Ed25519, etc.)

## Privacy-Preserving Features

### 1. Anonymous Deal Discovery

Users browse marketplace deals without revealing:
- Identity
- Location (beyond user-configured granularity)
- Browsing history
- Purchase intent

See [Deal Discovery](../marketplace/discovery.md) for details.

### 2. Ephemeral Signals

When users opt into personalized deals:
- Signals rotate every 1-6 hours
- Location is coarsened (district/city level)
- LTV shown as tier (high/medium/low), not exact amount
- No persistent tracking

See [Signal System](../marketplace/matching/signals.md) for details.

### 3. Privacy Proofs

Users can prove eligibility without revealing data:
- LTV threshold proofs (zkSNARK)
- Loyalty tier proofs (commitment schemes)
- Proximity proofs (without exact location)

See [Privacy-Preserving Proofs](../marketplace/discovery/proofs.md) for details.

### 4. Vendor Blindness

Vendors never see:
- Individual user identities
- Agent IDs
- Exact purchase amounts (only totals)
- Individual browsing behavior

See [Bazaar Architecture](../marketplace/bazaar.md) for details.

## Privacy by Use Case

### P2P Payments (User → User)

```
Alice → [PER Transfer in TEE] → Bob
```

**Privacy:**
- ✅ Amount hidden from blockchain
- ✅ Sender/recipient only known to participants
- ✅ No public transaction
- ✅ Even Sen cannot see transfer

### Merchant Payments (User → Vendor)

```
Alice → [PER Transfer] → Sen Treasury → [Batch Settlement] → Vendor
```

**Privacy:**
- ✅ Alice's payment is private
- ✅ Vendor doesn't see Alice's identity
- ❌ Vendor receives batched settlement (public on L1)
- ✅ Individual payments stay private

### AI Shopping (Agent → Vendor)

```
Agent browses anonymously → Evaluates locally → Negotiates privately → Pays via PER
```

**Privacy:**
- ✅ Browsing is anonymous
- ✅ Vendor learns of interest only when deal struck
- ✅ AI decisions happen in TEE
- ✅ No tracking across sessions

## Compliance & Regulation

Sen's privacy architecture is designed to balance user privacy with regulatory requirements:

### KYC/AML Integration

- **Identity verification** at onboarding (via Privy)
- **Transaction monitoring** for suspicious patterns
- **Audit trails** available to authorized regulators
- **User data requests** handled via legal process

### Privacy ≠ Anonymity

Sen provides **privacy** (protection from surveillance) not **anonymity** (complete untraceability):

- Users are known to Sen at onboarding
- Privacy protects users from vendors and third parties
- Law enforcement can access data via proper channels
- Designed for legitimate commerce, not illicit activity

## Future Enhancements

### Short-term (3-6 months)

- **Stealth addresses** for enhanced recipient privacy
- **Ring signatures** for deposit/withdrawal obfuscation
- **Decoy transactions** to obfuscate on-chain patterns

### Long-term (6-12 months)

- **Multi-party computation (MPC)** for decentralized key management
- **Fully homomorphic encryption (FHE)** for encrypted computation
- **Zero-knowledge proofs** for all privacy-sensitive operations
- **Cross-chain privacy** bridges

## Best Practices

For developers integrating with Sen:

1. **Never log sensitive data** - Payment amounts, balances, keys
2. **Use provided SDKs** - Encryption handled correctly
3. **Verify TEE attestations** - Ensure agents run in real TEEs
4. **Minimize data collection** - Only collect what's necessary
5. **User consent** - Always ask before sharing any data

For users:

1. **Protect backup phrases** - Store securely offline
2. **Configure signal granularity** - Choose location precision
3. **Review agent permissions** - Control what agents can do
4. **Monitor transactions** - Check activity regularly
5. **Report issues** - Contact support for suspicious activity

## Learn More

- [TEE Security Details](privacy/tee.md)
- [PER Architecture](privacy/per.md)
- [Encryption & Key Management](privacy/encryption.md)
- [Security Threat Model](../security/threat-model.md)
- [Privacy Guarantees](../security/privacy-guarantees.md)
