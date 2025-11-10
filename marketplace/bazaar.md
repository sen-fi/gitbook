# Bazaar Marketplace Architecture

The **Bazaar** (also called **Agora**) is Sen's privacy-preserving marketplace where vendors post deals and autonomous agents discover them - without surveillance, tracking, or identity exposure.

## Core Philosophy

**Vendor Blindness** - Vendors post deals publicly but remain completely blind to:
- Individual user identities
- Agent IDs or device identifiers
- Browsing behavior or search patterns
- Individual purchase decisions (only see aggregates)

**Sen as Secure Middleman** - Sen operates as a trusted TEE intermediary:
- Vendors post deals to Sen's registry
- Users browse anonymously through Sen
- Matching happens in TEE (vendors cannot intercept)
- Vendors learn about users only when deals are struck

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                 Vendor (Public Interface)                    │
│  - Posts deals with targeting criteria                       │
│  - Views aggregate metrics only                              │
│  - Never sees individual users/agents                        │
└──────────────────┬──────────────────────────────────────────┘
                   │ HTTP API
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              Vendor Registry (Public)                        │
│  - Public bulletin board for deals                           │
│  - No authentication required                                │
│  - Anyone can browse anonymously                             │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              TEE Matching Engine (Private)                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Runs in Trusted Execution Environment             │   │
│  │  - Loads opt-in agent signals (encrypted)           │   │
│  │  - Matches deals to eligible agents                 │   │
│  │  - Vendors NEVER see matching logic                 │   │
│  │  - Pushes matches to agent queues                   │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────┬──────────────────────────────────────────┘
                   │ Encrypted notifications
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              User Agents (TEE Instances)                     │
│  - Receive matched deal notifications                        │
│  - Evaluate locally                                          │
│  - Initiate purchase when ready                              │
└─────────────────────────────────────────────────────────────┘
```

## Component Architecture

### 1. Vendor Registry (Public)

Public API for vendors to post deals and browse metrics.

**Technology**: Express.js API + EventEmitter

**Endpoints**:
- `POST /deals` - Post new deal
- `GET /deals` - Browse public deals (no auth)
- `GET /deals/:dealId` - Get deal details
- `PUT /deals/:dealId` - Update deal
- `DELETE /deals/:dealId` - Remove deal
- `GET /metrics/:vendorId` - View aggregate metrics

**Event Emission**:
```typescript
// When vendor posts deal, emit event
registry.emit('deal.posted', {
  dealId: 'deal_abc123',
  vendorId: 'highlands-coffee',
  targeting: {
    location: { areas: ['Quận 1', 'Quận 3'] },
    ltv: { tier: 'high' },
    loyalty: { minLevel: 2 }
  }
});
```

### 2. Agent Signal Store (Ephemeral)

Privacy-preserving signals that users opt into for personalized deals.

**Technology**: Encrypted SQLite in TEE

**Signal Structure**:
```typescript
interface AgentSignal {
  // Rotates every 1-6 hours (no persistent tracking)
  ephemeralId: string;

  // Privacy-preserving signals
  ltv: {
    tier: 'high' | 'medium' | 'low'; // NOT exact amount
  };

  loyalty: {
    level: number; // 0-5 scale
  };

  location: {
    granularity: 'district' | 'ward' | 'city'; // User-configured
    area: string; // e.g., "Quận 1" (many users per area)
  };

  // User preferences
  preferences: {
    categories: string[];
    maxPrice?: number;
  };

  // Privacy: userId encrypted, only for internal matching
  encryptedUserId: Buffer;
}
```

**Key Features**:
- **Ephemeral IDs** rotate frequently (prevents tracking)
- **Coarse location** (district/city level, not GPS)
- **Tiered LTV** (high/medium/low, not exact dollars)
- **Opt-in only** (users control signal sharing)

See [Signal System](matching/signals.md) for details.

### 3. TEE Matching Engine (Private)

Event-driven worker that matches deals to agents in hardware TEE.

**Technology**: AWS Nitro Enclave / Intel TDX

**Matching Flow**:

```typescript
// 1. Vendor posts deal → Event emitted
registry.on('deal.posted', async (deal) => {
  // 2. TEE worker wakes up
  const matcher = new TEEMatchingEngine();

  // 3. Load opt-in signals (encrypted, in TEE)
  const signals = await matcher.loadAgentSignals();

  // 4. Match deal to eligible agents (IN TEE - vendor blind)
  const matches = signals.filter(signal => {
    // LTV match
    if (deal.requirements?.minLTV) {
      if (!meetsLTVThreshold(signal.ltv.tier, deal.requirements.minLTV)) {
        return false;
      }
    }

    // Loyalty match
    if (deal.requirements?.loyaltyTier) {
      if (signal.loyalty.level < mapTierToLevel(deal.requirements.loyaltyTier)) {
        return false;
      }
    }

    // Location match
    if (deal.targeting?.location) {
      if (!deal.targeting.location.areas.includes(signal.location.area)) {
        return false;
      }
    }

    return true;
  });

  // 5. Push notifications to matched agents (encrypted)
  await matcher.notifyAgents(matches, deal);

  // 6. Update vendor metrics (aggregate only)
  await updateVendorMetrics(deal.vendorId, {
    agentsMatched: matches.length, // Count only, no IDs
  });
});
```

**Privacy Guarantees**:
- ✅ Matching logic runs in TEE (vendor cannot see)
- ✅ Vendor gets aggregate count only
- ✅ Individual agent IDs never exposed
- ✅ Signal data encrypted at rest

See [TEE Matching](matching/tee-matching.md) for details.

### 4. User Agents (TEE Instances)

Receive matched deals and evaluate locally.

**Local Evaluation**:
```typescript
// Agent receives deal notification
agent.on('deal.matched', async (deal) => {
  // Evaluate locally (no network call)
  const eligible = await agent.evaluateEligibility(deal, {
    ltvThreshold: deal.requirements?.minLTV,
    loyaltyTier: deal.requirements?.loyaltyTier,
    proximityRequired: deal.requirements?.proximityRequired,
  });

  if (eligible) {
    // Show to user OR auto-purchase if criteria met
    if (shouldAutoPurchase(deal)) {
      await agent.initiatePurchase({ deal, paymentMethod: 'per' });
    } else {
      await showDealToUser(deal);
    }
  }
});
```

## Privacy Flow Breakdown

### Anonymous Browsing Phase

```
User Agent → Vendor Registry (GET /deals)
  ↓
Public deal listings (no auth, no logging)
  ↓
Agent evaluates locally → No network traffic
```

**Privacy**:
- ✅ No authentication required
- ✅ No cookies or tracking
- ✅ No IP logging
- ✅ Browsing stays local

### Matching Phase (Opt-in Signals)

```
Vendor posts deal
  ↓
Event emitted → TEE Matching Engine wakes
  ↓
Loads encrypted signals from TEE database
  ↓
Matches deals to agents (IN TEE)
  ↓
Sends encrypted notifications
  ↓
Updates vendor metrics (aggregate only)
```

**Privacy**:
- ✅ Signals encrypted at rest
- ✅ Matching happens in TEE
- ✅ Vendor sees count only
- ✅ No agent IDs exposed

### Purchase Phase

```
Agent decides to buy
  ↓
Generates proofs (LTV, loyalty, proximity)
  ↓
Sends DealAcceptance to Vendor (FIRST contact)
  ↓
Vendor confirms order
  ↓
Payment via PER (private)
```

**Privacy**:
- ✅ Vendor learns about user only now
- ✅ Proofs reveal minimum information
- ✅ Payment is private (PER)
- ✅ No linkage to browsing history

## What Vendors See

### Deal Performance Metrics (Aggregate Only)

```typescript
interface VendorMetrics {
  // Counts only, no identities
  agentsMatched: number;       // e.g., 23
  impressionsSent: number;     // e.g., 18 (push notifications)
  redemptions: number;         // e.g., 5 (completed purchases)
  conversionRate: number;      // e.g., 21.7% (redemptions/matched)

  // Regional breakdown (still no individual users)
  byLocation: {
    "Quận 1": { matched: 15, redemptions: 3 },
    "Quận 3": { matched: 8, redemptions: 2 }
  };

  // Time series (aggregate)
  byDay: {
    "2024-11-10": { matched: 10, redemptions: 2 },
    "2024-11-11": { matched: 13, redemptions: 3 }
  };

  // NO individual user data
  // NO agent IDs
  // NO redemption details (amounts, timestamps, etc.)
}
```

### What Vendors NEVER See

❌ **Individual user identities** - Encrypted user IDs never exposed
❌ **Agent IDs** - Ephemeral IDs rotate frequently
❌ **Browsing behavior** - Anonymous API, no logging
❌ **Individual redemptions** - Only aggregate counts
❌ **Exact locations** - Only coarse areas (district/city)
❌ **Exact LTV amounts** - Only tier (high/medium/low)
❌ **Purchase amounts** - Only that payment was received

## Targeting Options

Vendors can target deals using these criteria:

### 1. Location Targeting

```typescript
targeting: {
  location: {
    granularity: 'district',
    areas: ['Quận 1', 'Quận 3', 'Quận 5']
  }
}
```

**Privacy**: Areas are large (thousands of users), no GPS coordinates

### 2. LTV Targeting

```typescript
requirements: {
  minLTV: 100  // $100/month in category
}
```

**Privacy**: Agent provides tier (high/medium/low) OR zkSNARK proof, never exact amount

### 3. Loyalty Targeting

```typescript
requirements: {
  loyaltyTier: 'loyal',  // champion | loyal | regular | switcher
  category: 'coffee'
}
```

**Privacy**: Commitment-based proof, doesn't reveal exact purchase history

### 4. Proximity Targeting

```typescript
requirements: {
  proximityRequired: true,
  proximityRadius: 2000,  // 2km
  location: {
    lat: 10.7769,
    lng: 106.7009
  }
}
```

**Privacy**: Agent proves proximity without revealing exact GPS coordinates

## Deal Lifecycle

### 1. Deal Creation

```typescript
const deal = await bazaar.postDeal({
  vendorId: 'highlands-coffee',
  service: {
    type: 'product',
    category: 'coffee',
    description: 'Premium Vietnamese Coffee - Cà Phê Sữa Đá'
  },
  pricing: {
    basePrice: 5.00,
    discountedPrice: 4.00,
    discountPercent: 20,
    currency: 'USD'
  },
  requirements: {
    minLTV: 50,
    loyaltyTier: 'regular',
    proximityRequired: true,
    proximityRadius: 2000
  },
  availability: {
    inStock: true,
    maxRedemptions: 100,
    currentRedemptions: 0
  },
  expiresAt: Date.now() + 7 * 24 * 60 * 60 * 1000  // 7 days
});
```

### 2. Matching & Notification

```typescript
// TEE matching engine (automatic)
const matched = await matchingEngine.processDeal(deal);

// Vendor receives aggregate metrics
console.log(`Matched to ${matched.count} agents`);
// Vendor does NOT receive agent IDs
```

### 3. User Purchase

```typescript
// User agent (autonomous OR user-approved)
const confirmation = await agent.initiatePurchase({
  deal,
  paymentMethod: 'per',
  includeProofs: true
});

// Vendor receives order confirmation (FIRST time learning about user)
```

### 4. Settlement

```typescript
// Payment via PER (private)
await agent.executePayment({
  recipient: vendorWallet,
  amount: deal.pricing.discountedPrice,
  method: 'per'
});

// Vendor receives batched settlement from Sen treasury
```

## Security Considerations

### Sybil Resistance

Prevent users from creating fake agents to game system:

1. **Proof of Stake** - Agents must lock collateral
2. **Reputation Scoring** - Track agent behavior over time
3. **TEE Attestation** - Only genuine TEE instances accepted
4. **Rate Limiting** - Limit deal redemptions per user/agent

### Vendor Trust

Vendors must trust Sen's TEE matching:

1. **Open Source** - All matching logic auditable
2. **TEE Attestation** - Vendors can verify code integrity
3. **Cryptographic Proofs** - Vendors can verify match counts
4. **Third-party Audits** - Regular security audits

### Privacy Leaks

Potential privacy leaks and mitigations:

| Risk | Mitigation |
|------|------------|
| **Location deanonymization** | Coarse granularity (district vs GPS) |
| **LTV inference** | Tier-based (high/medium/low) |
| **Tracking via timing** | Ephemeral IDs rotate frequently |
| **Cross-vendor tracking** | No shared identifiers across vendors |
| **Vendor collusion** | Separate ephemeral IDs per vendor |

## Performance Characteristics

| Metric | Target | Notes |
|--------|--------|-------|
| Deal posting | <100ms | Write to registry |
| Matching latency | <500ms | TEE processing |
| Notification delivery | <2s | Push to agents |
| Browse deals | <100ms | Public API (cached) |
| Local evaluation | <10ms | No network call |

## Scaling Strategy

### Horizontal Scaling

```
                  Load Balancer
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  TEE Instance 1  TEE Instance 2  TEE Instance 3
  (User agents)   (User agents)   (User agents)
        │              │              │
        └──────────────┼──────────────┘
                       ▼
               Redis Event Queue
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  Matcher 1       Matcher 2       Matcher 3
  (TEE workers)   (TEE workers)   (TEE workers)
```

**Scaling Dimensions**:
1. **User agents** - Add more TEE instances (stateless)
2. **Matching workers** - Add more TEE workers (event-driven)
3. **Registry** - Cache with Redis, replicate read-only

See [Multi-Tenant Scaling](../architecture/agents/scaling.md) for details.

## Future Enhancements

### Short-term (3-6 months)

- **zkSNARK proofs** for LTV (fully zero-knowledge)
- **Ring signatures** for proximity (k-anonymity)
- **Reputation system** with privacy (commitments)
- **Cross-vendor deals** (bundle deals across vendors)

### Long-term (6-12 months)

- **Decentralized registry** (IPFS + blockchain)
- **Multi-party computation** for matching (no single TEE)
- **Homomorphic encryption** for encrypted compute
- **Global marketplace** (cross-border deals)

## Learn More

- [Deal Discovery](discovery.md)
- [Signal System](matching/signals.md)
- [TEE Matching](matching/tee-matching.md)
- [Vendor Integration](vendor-integration.md)
- [Privacy Guarantees](../security/privacy-guarantees.md)
