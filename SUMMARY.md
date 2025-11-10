# Table of contents

* [Introduction](README.md)

## Architecture

* [Privacy Architecture](architecture/privacy.md)
  * [Trusted Execution Environments](architecture/privacy/tee.md)
  * [Private Ephemeral Rollups](architecture/privacy/per.md)
  * [Encryption & Key Management](architecture/privacy/encryption.md)
* [Agent Infrastructure](architecture/agents.md)
  * [Event-Driven Design](architecture/agents/event-driven.md)
  * [TEE-Hosted Agents](architecture/agents/tee-agents.md)
  * [Multi-Tenant Scaling](architecture/agents/scaling.md)
* [Payment System](architecture/payments.md)
  * [PER Transfers](architecture/payments/per-transfers.md)
  * [L1 Settlement](architecture/payments/l1-settlement.md)
  * [Payment Router](architecture/payments/router.md)

## X.402 Protocol

* [Protocol Overview](protocol/overview.md)
* [Message Types](protocol/messages.md)
  * [Quote Request](protocol/messages/quote-request.md)
  * [Quote Response](protocol/messages/quote-response.md)
  * [Counter Offer](protocol/messages/counter-offer.md)
  * [Deal Acceptance](protocol/messages/deal-acceptance.md)
  * [Deal Confirmation](protocol/messages/deal-confirmation.md)
* [Payment Flow](protocol/payment-flow.md)
* [Privacy Guarantees](protocol/privacy.md)
* [Negotiation Strategies](protocol/negotiation.md)

## Bazaar Marketplace

* [Bazaar Architecture](marketplace/bazaar.md)
* [Deal Discovery](marketplace/discovery.md)
  * [Anonymous Browsing](marketplace/discovery/anonymous-browsing.md)
  * [Local Eligibility Evaluation](marketplace/discovery/eligibility.md)
  * [Privacy-Preserving Proofs](marketplace/discovery/proofs.md)
* [Matching Engine](marketplace/matching.md)
  * [Signal System](marketplace/matching/signals.md)
  * [TEE Matching](marketplace/matching/tee-matching.md)
  * [Vendor Metrics](marketplace/matching/metrics.md)
* [Vendor Integration](marketplace/vendor-integration.md)

## AI Agent System

* [Hybrid AI Architecture](ai/architecture.md)
* [Research Agent](ai/research.md)
* [Decision Agent](ai/decision.md)
* [Negotiation Agent](ai/negotiation.md)
* [Substitution Agent](ai/substitution.md)
* [Session Memory](ai/memory.md)

## API Reference

* [Client SDK](api/client-sdk.md)
  * [Installation](api/client-sdk/installation.md)
  * [Authentication](api/client-sdk/authentication.md)
  * [Configuration](api/client-sdk/configuration.md)
* [Agent API](api/agent-api.md)
  * [Negotiator](api/agent-api/negotiator.md)
  * [Discovery](api/agent-api/discovery.md)
  * [Payment Execution](api/agent-api/payment.md)
* [Vendor API](api/vendor-api.md)
  * [Deal Posting](api/vendor-api/deal-posting.md)
  * [Quote Handling](api/vendor-api/quote-handling.md)
  * [Settlement](api/vendor-api/settlement.md)

## Security

* [Threat Model](security/threat-model.md)
* [TEE Security](security/tee-security.md)
* [Privacy Guarantees](security/privacy-guarantees.md)
* [Attack Scenarios](security/attack-scenarios.md)
* [Best Practices](security/best-practices.md)

## Deployment

* [Infrastructure Overview](deployment/overview.md)
* [AWS Nitro Setup](deployment/aws-nitro.md)
* [Solana Configuration](deployment/solana.md)
* [Secrets Management](deployment/secrets.md)
* [Monitoring](deployment/monitoring.md)

## Examples

* [Basic P2P Transfer](examples/p2p-transfer.md)
* [Agent Negotiation](examples/negotiation.md)
* [Vendor Integration](examples/vendor-integration.md)
* [Privacy Proofs](examples/privacy-proofs.md)

## Resources

* [Glossary](resources/glossary.md)
* [FAQ](resources/faq.md)
* [Contributing](resources/contributing.md)
* [Changelog](resources/changelog.md)
