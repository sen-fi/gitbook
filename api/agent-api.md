# Agent API Reference

The Agent API provides the core functionality for autonomous agents to discover deals, negotiate with vendors, and execute payments on behalf of users.

## Installation

```bash
npm install @sen-protocol/sdk
# or
yarn add @sen-protocol/sdk
# or
bun add @sen-protocol/sdk
```

## Quick Start

```typescript
import {
  X402Negotiator,
  DealDiscovery,
  createPrivatePaymentsClient
} from '@sen-protocol/sdk';

// Initialize agent components
const negotiator = new X402Negotiator({
  userId: 'user_abc123',
  agentId: 'agent_xyz789',
  config: {
    autoNegotiate: true,
    maxNegotiationRounds: 3,
    minSavingsToNegotiate: 5
  }
});

const discovery = new DealDiscovery({
  registryUrl: 'https://bazaar.sen.fi'
});

// Browse deals anonymously
const deals = await discovery.browseDeals({
  category: 'coffee',
  maxPrice: 10
});

// Initiate purchase
const confirmation = await negotiator.initiatePurchase({
  deal: deals[0],
  paymentMethod: 'per',
  includeProofs: true
});
```

## Core Components

### X402Negotiator

The negotiation engine for agent-to-agent communication.

**Constructor**

```typescript
new X402Negotiator(params: {
  userId: string;
  agentId: string;
  config: NegotiatorConfig;
  vendorClients: Map<string, VendorClient>;
  persistence: StatePersistence;
  loyalty: LoyaltyProgramManager;
  ltvScorer: LTVScorer;
})
```

**Configuration**

```typescript
interface NegotiatorConfig {
  // Negotiation strategy
  autoNegotiate: boolean;          // Auto-counter if beneficial
  maxNegotiationRounds: number;    // Max rounds (default: 3)
  minSavingsToNegotiate: number;   // Min $ to negotiate (default: $5)

  // Comparison
  minVendorsToCompare: number;     // Quote from N vendors (default: 2)
  comparisonTimeoutMs: number;     // Timeout for quotes (default: 10s)

  // Privacy
  shareSignals: boolean;           // Share LTV/loyalty (default: true)
  shareLocation: boolean;          // Share proximity (default: false)
}
```

**Methods**

#### initiatePurchase()

Privacy-first purchase from discovered deal.

```typescript
async initiatePurchase(params: {
  deal: VendorDealPosting;
  paymentMethod?: 'per' | 'external';
  includeProofs?: boolean;
}): Promise<DealConfirmation>
```

**Example**

```typescript
const confirmation = await negotiator.initiatePurchase({
  deal: selectedDeal,
  paymentMethod: 'per',
  includeProofs: true  // Include LTV/loyalty proofs
});

console.log(`Order confirmed: ${confirmation.orderId}`);
console.log(`Tracking: ${confirmation.trackingInfo}`);
```

#### initiateNegotiation()

Start negotiation with counter-offer.

```typescript
async initiateNegotiation(params: {
  deal: VendorDealPosting;
  proposedPrice: number;
  reason: string;
  includeProofs?: boolean;
}): Promise<NegotiationSession>
```

**Example**

```typescript
const session = await negotiator.initiateNegotiation({
  deal: selectedDeal,
  proposedPrice: 4.50,  // Counter-offer $4.50 vs $5.00
  reason: 'Loyal customer with high LTV',
  includeProofs: true
});

if (session.status === 'quoted') {
  console.log(`Negotiation successful! Final price: $${session.finalPrice}`);
}
```

#### compareVendors()

Request quotes from multiple vendors and compare.

```typescript
async compareVendors(params: {
  vendorIds: string[];
  service: QuoteRequest['service'];
  preferences?: QuoteRequest['preferences'];
}): Promise<MultiVendorComparison>
```

**Example**

```typescript
const comparison = await negotiator.compareVendors({
  vendorIds: ['highlands', 'phuclong', 'thecoffeehouse'],
  service: {
    type: 'product',
    category: 'coffee',
    description: 'Vietnamese iced coffee'
  }
});

console.log(`Best price: ${comparison.bestPrice.vendorId} - $${comparison.bestPrice.price}`);
console.log(`Best value: ${comparison.bestValue.vendorId} (score: ${comparison.bestValue.score})`);
console.log(`Savings: $${comparison.recommended.savings}`);
```

#### requestQuote()

Request single vendor quote (legacy flow).

```typescript
async requestQuote(params: {
  vendorId: string;
  service: QuoteRequest['service'];
  preferences?: QuoteRequest['preferences'];
}): Promise<NegotiationSession>
```

#### counterOffer()

Send counter-offer during negotiation.

```typescript
async counterOffer(params: {
  sessionId: string;
  proposedPrice?: number;
  proposedQuantity?: number;
  reason?: string;
}): Promise<void>
```

#### acceptOffer()

Accept vendor's offer.

```typescript
async acceptOffer(sessionId: string): Promise<DealConfirmation>
```

#### getSession()

Retrieve negotiation session by ID.

```typescript
getSession(sessionId: string): NegotiationSession | null
```

### DealDiscovery

Anonymous deal browsing and local evaluation.

**Constructor**

```typescript
new DealDiscovery(params?: {
  registryUrl?: string;
  cache?: CacheOptions;
})
```

**Methods**

#### browseDeals()

Browse deals anonymously (no auth required).

```typescript
async browseDeals(filter?: {
  category?: string;
  maxPrice?: number;
  minPrice?: number;
  requiresProximity?: boolean;
  vendorIds?: string[];
  tags?: string[];
  negotiableOnly?: boolean;
  sortBy?: 'price' | 'discount' | 'newest' | 'ending_soon';
  limit?: number;
}): Promise<VendorDealPosting[]>
```

**Example**

```typescript
// Find cheap coffee deals near me
const deals = await discovery.browseDeals({
  category: 'coffee',
  maxPrice: 5,
  requiresProximity: true,
  sortBy: 'discount',
  limit: 10
});

console.log(`Found ${deals.length} deals`);
deals.forEach(deal => {
  console.log(`${deal.vendorName}: $${deal.pricing.discountedPrice || deal.pricing.basePrice}`);
});
```

#### getDeal()

Get specific deal by ID.

```typescript
async getDeal(dealId: string): Promise<VendorDealPosting | null>
```

#### evaluateEligibility()

Evaluate if user is eligible for deal (local evaluation, no network call).

```typescript
evaluateEligibility(
  deal: VendorDealPosting,
  userProfile: {
    ltvByCategory: Map<string, number>;
    loyaltyCards: LoyaltyCard[];
    location?: { lat: number; lng: number };
  }
): { eligible: boolean; reason?: string }
```

**Example**

```typescript
const eligible = discovery.evaluateEligibility(deal, {
  ltvByCategory: new Map([['coffee', 150]]),  // $150/month on coffee
  loyaltyCards: [{ vendorId: 'highlands', tier: 'gold' }],
  location: { lat: 10.7769, lng: 106.7009 }
});

if (eligible.eligible) {
  console.log('You qualify for this deal!');
} else {
  console.log(`Not eligible: ${eligible.reason}`);
}
```

#### subscribeToUpdates()

Subscribe to deal updates (new deals, price changes, expiring deals).

```typescript
subscribeToUpdates(
  callback: (event: DealEvent) => void,
  filter?: DealFilter
): Subscription
```

**Example**

```typescript
const subscription = discovery.subscribeToUpdates(
  (event) => {
    if (event.type === 'deal.new') {
      console.log(`New deal: ${event.deal.vendorName} - $${event.deal.pricing.discountedPrice}`);
    }
  },
  { category: 'coffee', maxPrice: 10 }
);

// Later: unsubscribe
subscription.unsubscribe();
```

### PrivatePaymentsClient

Client for executing private payments via MagicBlock PER.

**Constructor**

```typescript
new PrivatePaymentsClient(params: {
  connection: Connection;          // Solana devnet connection
  ephemeralConnection: Connection; // PER TEE connection
  wallet: Wallet;                  // User's wallet
  programId: PublicKey;            // Private payments program
})
```

**Methods**

#### initializeDeposit()

Initialize deposit account for user.

```typescript
async initializeDeposit(
  tokenMint: PublicKey
): Promise<string>
```

#### deposit()

Deposit tokens to PER.

```typescript
async deposit(
  tokenMint: PublicKey,
  amount: number
): Promise<string>
```

#### delegate()

Delegate deposit to PER TEE.

```typescript
async delegate(
  tokenMint: PublicKey
): Promise<string>
```

#### privateTransfer()

Execute private transfer on PER (TEE).

```typescript
async privateTransfer(
  recipient: PublicKey,
  tokenMint: PublicKey,
  amount: number
): Promise<string>
```

#### undelegate()

Remove deposit from PER.

```typescript
async undelegate(
  tokenMint: PublicKey
): Promise<string>
```

#### withdraw()

Withdraw tokens from deposit.

```typescript
async withdraw(
  tokenMint: PublicKey,
  amount: number
): Promise<string>
```

#### getDepositBalance()

Check deposit balance.

```typescript
async getDepositBalance(
  tokenMint: PublicKey
): Promise<number>
```

#### isDelegated()

Check if deposit is delegated to PER.

```typescript
async isDelegated(
  tokenMint: PublicKey
): Promise<boolean>
```

**Example: Complete Payment Flow**

```typescript
import { Connection, PublicKey } from '@solana/web3.js';

const client = new PrivatePaymentsClient({
  connection: new Connection('https://api.devnet.solana.com'),
  ephemeralConnection: new Connection('https://tee.magicblock.app'),
  wallet: userWallet,
  programId: new PublicKey('EnhkomtzKms55jXi3ijn9XsMKYpMT4BJjmbuDQmPo3YS')
});

const USDC_MINT = new PublicKey('4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU');

// 1. Initialize deposit (one-time)
await client.initializeDeposit(USDC_MINT);

// 2. Deposit tokens
await client.deposit(USDC_MINT, 100);  // Deposit $100

// 3. Delegate to PER
await client.delegate(USDC_MINT);

// 4. Private transfer (in TEE, invisible to blockchain)
await client.privateTransfer(
  vendorPublicKey,
  USDC_MINT,
  5  // Pay $5
);

// 5. Check remaining balance
const balance = await client.getDepositBalance(USDC_MINT);
console.log(`Remaining balance: $${balance}`);
```

## Type Definitions

### VendorDealPosting

```typescript
interface VendorDealPosting {
  dealId: string;
  vendorId: string;
  vendorName: string;
  timestamp: number;
  expiresAt: number;

  service: {
    type: 'product' | 'service' | 'subscription';
    category: string;
    description: string;
    quantity?: number;
  };

  pricing: {
    basePrice: number;
    discountedPrice?: number;
    discountPercent?: number;
    currency: string;
  };

  requirements?: {
    minLTV?: number;
    category?: string;
    loyaltyTier?: 'champion' | 'loyal' | 'regular' | 'switcher';
    proximityRequired?: boolean;
    proximityRadius?: number;
    location?: {
      lat: number;
      lng: number;
      name?: string;
    };
  };

  availability: {
    inStock: boolean;
    quantity?: number;
    maxRedemptions?: number;
    currentRedemptions: number;
    estimatedDelivery?: number;
  };

  paymentOptions: {
    acceptsPER: boolean;
    acceptsExternal: boolean;
    acceptsCrypto?: string[];
  };

  negotiable: boolean;
  tags?: string[];
}
```

### DealConfirmation

```typescript
interface DealConfirmation {
  confirmationId: string;
  acceptanceId: string;
  timestamp: number;
  confirmed: boolean;

  paymentDetails: {
    recipient: string;
    amount: number;
    tokenMint?: string;
    memo?: string;
    invoiceUrl?: string;
  };

  orderId?: string;
  estimatedCompletion?: number;
  trackingInfo?: string;

  x402SessionId?: string;
  x402TxHash?: string;
}
```

### NegotiationSession

```typescript
interface NegotiationSession {
  sessionId: string;
  userId: string;
  vendorId: string;

  startedAt: number;
  lastActivityAt: number;
  completedAt?: number;

  status: 'requesting' | 'quoted' | 'negotiating' | 'accepted' | 'confirmed' | 'paid' | 'failed';

  quoteRequest?: QuoteRequest;
  quoteResponse?: QuoteResponse;
  counterOffers: CounterOffer[];
  counterOfferResponses: CounterOfferResponse[];
  acceptance?: DealAcceptance;
  confirmation?: DealConfirmation;

  finalPrice?: number;
  txSignature?: string;
}
```

## Error Handling

All SDK methods throw typed errors:

```typescript
import { Sen402Error, NetworkError, ValidationError } from '@sen-protocol/sdk';

try {
  const confirmation = await negotiator.initiatePurchase({ deal });
} catch (error) {
  if (error instanceof Sen402Error) {
    console.error(`X.402 error: ${error.code} - ${error.message}`);
  } else if (error instanceof NetworkError) {
    console.error(`Network error: ${error.message}`);
  } else if (error instanceof ValidationError) {
    console.error(`Validation error: ${error.field} - ${error.message}`);
  }
}
```

**Error Codes**

| Code | Description |
|------|-------------|
| `DEAL_EXPIRED` | Deal has expired |
| `DEAL_SOLD_OUT` | Deal max redemptions reached |
| `INSUFFICIENT_BALANCE` | User has insufficient funds |
| `PROOF_VERIFICATION_FAILED` | Privacy proof verification failed |
| `VENDOR_UNAVAILABLE` | Vendor agent is offline |
| `NEGOTIATION_REJECTED` | Vendor rejected counter-offer |
| `PAYMENT_FAILED` | Payment execution failed |

## Examples

See [Examples](../examples/) for complete code samples:

- [Basic P2P Transfer](../examples/p2p-transfer.md)
- [Agent Negotiation](../examples/negotiation.md)
- [Privacy Proofs](../examples/privacy-proofs.md)

## Learn More

- [X.402 Protocol Overview](../protocol/overview.md)
- [Payment Flow](../protocol/payment-flow.md)
- [Client SDK](client-sdk.md)
- [Vendor API](vendor-api.md)
