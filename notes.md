# stuff

## Presentation requirements

- its main ideas
- technical concepts and terminology in simple language
- how the proposed engineering solution works
- the architectural decisions and why they were made
- diagrams, examples, or illustrations where they help explain complex concepts
- your own understanding rather than simply repeating the original article

*The main objective is not to create a complicated presentation. It is to make complicated engineering concepts as easy to understand as possible.*

## about the company

Stripe is an Irish-American financial technology company that provides payment processing and financial infrastructure for online and in-person businesses.

## Presentation

### its main ideas

In the early 2010s, accepting online payments required complex bank approvals, merchant accounts, and cumbersome integration. The Collisons proposed a simple code snippet (about seven lines) that developers could embed into a website or app to accept cards.

### Technical concepts and terminology

the Collison brothers showed how a developer could charge a card in a single API call without merchant applications, underwriting waits, or heavy SDKs:
```cmd
curl https://api.stripe.com/v1/charges \
  -u sk_test_xxx: \
  -d amount=2000 \
  -d currency=usd \
  -d source=tok_visa \
  -d description="Test charge"
```

### High-level architecture

(jpg)

### Low-level architectural decision

The system provides a number of key architectural principles to ensure a "seven-line" integration is safe at global scale: Client-Side Tokenization (Isolating PCI Scope), Unified State Machine (PaymentIntents), Layer-Wide Idempotency, Immutable Double-Entry Ledger, Event-Driven Asynchronous Pipeline.

#### Client-Side Tokenization (Isolating PCI Scope)

When a store stores your credit card number, then hackers can steal it.

You don't enter your card number into the store. Instead, you enter it in to a secure, tamper proof window that sends it right to a digital bank vault. It is safely locked in the vault and the store receives a harmless "claim check" (token). If hackers get into the store's system, they will only discover worthless claim checks, not your actual card number.

Credit card numbers are never stored on the merchant's servers. Sensitive card information is sent directly to Stripe's isolated card vault from client-side libraries such as Stripe Elements.

Token Exchange: A non-sensitive token or PaymentMethod ID is returned by the vault. All downstream microservices only communicate with that token, and the merchant and most of Stripe's internal services are not confined within the strict limits of PCI DSS compliance.

#### Unified State Machine (PaymentIntents)

The payments can become messy (such as waiting for bank approval, 3D Secure pop-ups) causing confusion with glitches.

Each payment follows a tight, unalterable process like a package: Wait for Payment → Needs Approval → Processing → Success. The system physically stops impossible glitches from happening such as a payment going from "Failed" to "Success.

The initial iterations of Stripe's Charges object were based on the synchronous credit card model. Global payment methods (such as bank debits, regional wallets, and 3D Secure authentication) added multi-step, asynchronous delays, which forced Stripe to rethink the core of its product in the wake of these new features.

A PaymentIntent represents a payment as a strict state machine (requires_payment_method -> requires_action -> processing -> succeeded). It makes it harder to make a payment fail and then succeed, or to make a successful payment fail.

#### Layer-Wide Idempotency

You may be charged three times if your internet connection goes down and you click "Pay" three times.

Each time you pay, your device will create a different "receipt ID". When you run into the same ID again, it behaves like a smart bouncer who has already checked a ticket: "I have already processed the same request, here is your success message, and it will safely ignore duplicate clicks.

Timeouts, dropped connections occur regularly in financial networks. For mutating API calls, Stripe needs a header called Idempotency-Key that is sent by the client.

The API layer performs a check on the key before executing a charge, against an atomic store. When the request is already processed or is being processed, Stripe returns the previously cached result instead of double charging the customer.

#### Immutable Double-Entry Ledger

When a database simply changes a number from $50 to $40, it's easy to cover up errors or fraud.

The system functions similar to an old school accountant. Does not delete or overwrite anything. All transactions in money are recorded in permanent ink in pairs, "debit and credit.All money transactions are recorded in permanent ink in pairs "debit and credit. When a mistake is made, it is not deleted, it is replaced by a new entry which corrects the error. This results in an un-hackable history which can be audited at any time.

Stripe isn't just adding or subtracting a balance column on a database. All money transactions are entered as a double entry (debit and credit) in an unchangeable append-only ledger.

Records are never modified or erased (corrections are added as new entries), so the system provides an audit trail of all cryptographic activity which can be reconciled with the external acquiring banks and card networks daily.

#### Event-Driven Asynchronous Pipeline

You can spend time checking for fraud, sending receipts by email and updating your store analytics, which may slow down your checkout.

As soon as your payment is securely deposited, the primary system immediately passes the additional work over to a "background crew. When a server stalls or a network fails, the crew keeps the task alive and carries on with the background retries until the task succeeds, and you don't even notice that any time was lost.

After a core payment state transition happens in a transactional database, secondary state transitions (like fraud scoring - Stripe Radar, analytics, merchant notifications, etc.) occur asynchronously using an internal event bus, such as Kafka.

The transactional outbox pattern guarantees that events will be published reliably even if a worker crashes in the middle of its execution. Bank settlements and chargebacks may occur hours or days after the initial transaction, so Stripe will attempt to notify merchants by sending updates through webhooks that are signed with a cryptographic signature, and will retry with exponentially increasing delays.

### references

https://stripe.dev/blog/payment-api-design
https://stripe.com/payments/elements
https://scaledojo.dev/blogs/payment-processing-architecture-how-stripe-moves-money-safely
https://spacecomplexity.ai/blog/payment-system-design-interview
https://dev.to/sgchris/building-a-payment-system-stripes-architecture-for-financial-transactions-3mlg
https://tpbabparn.medium.com/designing-a-payment-backend-with-stripe-integration-in-practice-64bb10ad9af8
https://andreyka26.com/payment-gateway-stripe-system-design-with-microsoft-engineer


### Termins

curl | stripe | SDK | the internet archive wayback machine | ledger |














