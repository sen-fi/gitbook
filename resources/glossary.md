# Glossary

## A

**Agent**
An autonomous software entity that acts on behalf of a user to discover deals, negotiate with vendors, and execute payments.

**Agent Card**
A machine-readable document describing an agent's capabilities, similar to OpenAPI spec for REST APIs. Follows Google's Agent-to-Agent (A2A) specification.

**Agora**
See [Bazaar](#bazaar)

## B

**Bazaar**
Sen's privacy-preserving marketplace where vendors post deals and agents discover them anonymously. Also called "Agora".

## C

**Counter-Offer**
A proposal from a user agent to modify the terms of a vendor's quote, typically proposing a different price.

## D

**Deal**
An offer posted by a vendor on the Bazaar marketplace, including pricing, requirements, and availability.

**Deal Acceptance**
The message sent by a user agent to accept a vendor's offer and initiate purchase.

**Deal Confirmation**
The vendor's response confirming an order and providing payment details.

**Delegation**
The process of transferring control of a Solana deposit account to MagicBlock's PER validator, enabling private transfers in TEE.

**Deposit**
A Program Derived Address (PDA) on Solana that tracks a user's balance in the Private Payments program.

## E

**Ephemeral Connection**
The network connection to MagicBlock's TEE endpoint (`https://tee.magicblock.app`), used for private PER operations.

**Ephemeral ID**
A rotating identifier used in agent signals that prevents long-term tracking. Rotates every 1-6 hours.

**Event-Driven**
Architecture pattern where agents wake up on-demand in response to events (scheduled tasks, user actions, deal postings) rather than running 24/7.

## L

**L1**
Layer 1 blockchain (Solana mainnet or devnet), where final settlement occurs publicly.

**L2**
Layer 2 scaling solution built on top of L1. Sen uses MagicBlock PER as a privacy-focused L2.

**LTV (Lifetime Value)**
The total amount a user spends in a category over time. Used for deal targeting and personalized pricing.

**LTV Proof**
A cryptographic proof that a user's spending exceeds a threshold without revealing the exact amount.

## M

**MagicBlock**
Infrastructure provider for Private Ephemeral Rollups (PER) on Solana, using Intel TDX for TEE security.

**Matching Engine**
TEE-based service that matches vendor deals to eligible user agents based on targeting criteria, without exposing individual users to vendors.

## N

**Negotiation**
The back-and-forth process where user agents and vendor agents agree on deal terms (price, quantity, delivery).

**Negotiation Session**
A stateful interaction tracking the full negotiation lifecycle from quote request through payment.

## P

**PDA (Program Derived Address)**
A Solana account address derived deterministically from a program ID and seeds, used for deposit accounts.

**PER (Private Ephemeral Rollups)**
MagicBlock's L2 solution for private Solana transactions using Intel TDX TEEs. Transactions occur off-chain in encrypted state, with periodic L1 settlement.

**Privacy Proof**
A cryptographic proof allowing users to demonstrate eligibility for deals without revealing sensitive information (e.g., LTV threshold proof, loyalty proof, proximity proof).

## Q

**Quote**
A personalized offer from a vendor in response to a quote request, including pricing, discounts, and availability.

**Quote Request**
A message from a user agent requesting pricing from a vendor for a specific service or product.

## S

**Sen Score**
Credit scoring system for peer-to-peer lending (planned feature).

**Settlement**
The process of finalizing a payment on the blockchain (L1), making it immutable and publicly verifiable.

**Signal**
Privacy-preserving data shared by user agents for deal matching (LTV tier, loyalty level, location area). Ephemeral and coarse-grained.

**Signal Store**
Encrypted database in TEE storing opt-in user signals for deal matching.

## T

**TEE (Trusted Execution Environment)**
Hardware-isolated secure enclave (Intel TDX, AWS Nitro) that provides cryptographic guarantees for private computation.

**TEE Attestation**
Cryptographic proof that code is running in a genuine TEE with expected properties (integrity, confidentiality).

**Targeting**
Vendor-specified criteria for which users should see a deal (location, LTV, loyalty tier).

## V

**Vault**
A Program Derived Address holding a user's SPL tokens in the SenWealth automated portfolio management system.

**Vendor Agent**
An HTTP endpoint operated by a merchant that handles quote requests, negotiation, and order confirmations.

**Vendor Blindness**
Security property where vendors cannot see individual users, agents, or their behavior - only aggregate metrics.

**Vendor Registry**
Public bulletin board where vendors post deals. No authentication required for browsing.

## X

**X.402 Protocol**
Sen's agent-to-agent communication protocol for autonomous negotiation and payment, inspired by HTTP 402 Payment Required.

## Z

**Zero-Knowledge Privacy**
Privacy model where even the platform operator (Sen) cannot access user financial data without explicit consent.

**zkSNARK (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge)**
Cryptographic proof system allowing one party to prove they know something without revealing what they know. Used for privacy proofs.

---

## Acronyms

| Acronym | Full Name |
|---------|-----------|
| A2A | Agent-to-Agent |
| API | Application Programming Interface |
| AWS | Amazon Web Services |
| E2EE | End-to-End Encryption |
| FHE | Fully Homomorphic Encryption |
| HTTP | Hypertext Transfer Protocol |
| KYC | Know Your Customer |
| L1 | Layer 1 (blockchain) |
| L2 | Layer 2 (scaling solution) |
| LTV | Lifetime Value |
| MPC | Multi-Party Computation |
| P2P | Peer-to-Peer |
| PDA | Program Derived Address |
| PER | Private Ephemeral Rollups |
| SDK | Software Development Kit |
| SPL | Solana Program Library |
| TDX | Trust Domain Extensions (Intel) |
| TEE | Trusted Execution Environment |
| UX | User Experience |
| zkSNARK | Zero-Knowledge Succinct Non-Interactive Argument of Knowledge |

---

[Back to Resources](../resources/)
