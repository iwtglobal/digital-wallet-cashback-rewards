# Digital Wallet Cashback & Rewards

An educational guide to **digital wallet cashback**, **e-wallet rewards engines**, and wallet loyalty points. Written for fintech, telecom, and payment teams evaluating platforms in the [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/) family.

---

## What Is Digital Wallet Cashback?

**Digital wallet cashback** is a reward credited back to a customer's wallet after a qualifying action: a merchant payment, a bill payment, an airtime top-up, or a voucher purchase. An **e-wallet rewards engine** decides who qualifies, how much they earn, when the reward becomes spendable, and who funds it (the operator, the merchant, or a brand partner).

Cashback looks simple on a marketing banner. Underneath, it is a ledger problem. Every reward is a liability until it expires or is spent, every campaign has a budget that must not be overspent, and every reversal of the original payment has to claw back the reward it earned. Wallet programs that treat cashback as a side table instead of proper ledger postings end up with balances nobody can explain.

### Why a proper rewards engine matters

- **Adoption** — cashback is one of the most reliable levers for moving cash users onto a wallet  
- **Merchant acquisition** — co-funded campaigns give merchants a reason to accept wallet QR payments  
- **Budget control** — campaign caps stop a viral promotion from draining the marketing budget overnight  
- **Clean accounting** — reward liabilities, funders, and expiries stay visible to finance  

---

## Architecture Overview: Event → Reward Pipeline

```
Qualifying event (payment / top-up / bill / voucher purchase)
        │
        ▼
Campaign matcher (eligibility rules, segments, dates)
        │
        ▼
Budget and cap check (per user, per campaign, per funder)
        │
        ▼
Reward calculation (fixed, percentage, tiered, capped)
        │
        ▼
Pending reward ──► Maturity window ──► Spendable reward balance
        │
        ▼
Clawback on reversal / expiry sweep / finance export
```

### Core layers

| Layer | Responsibility |
|-------|----------------|
| **Event intake** | Consumes settled transactions with idempotent event IDs |
| **Campaign rules** | Segments, merchant lists, MCCs, date windows, minimum spend |
| **Budget ledger** | Tracks remaining campaign budget and per-user caps |
| **Reward ledger** | Separate reward sub-balance with pending and spendable states |
| **Clawback & expiry** | Reverses rewards tied to refunded payments; expires unused rewards |
| **Reporting** | Cost per acquisition, funder statements, liability reports |

See [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for a component view and suggested data fields.

---

## How Wallet Cashback Works, Step by Step

### 1. Define the campaign

Set the trigger (for example, first QR payment above a minimum amount), the reward formula, the funder, the total budget, the per-user cap, and the start and end dates.

### 2. Match settled events only

Evaluate rewards against settled transactions, not authorizations. Rewarding a payment that later fails is the most common source of cashback leakage.

### 3. Reserve budget, then credit a pending reward

Decrement the campaign budget atomically and credit the reward to a pending state. If the budget is exhausted, the event is recorded as "not rewarded: budget" so support can explain it.

### 4. Mature the reward

After a holding window (often aligned with the refund window), move the reward from pending to spendable. This is what makes clawback cheap.

### 5. Claw back, expire, and report

Refunds and reversals trigger a linked clawback posting. Unused rewards expire according to published terms, and finance receives funder-level statements.

---

## Common Campaign Patterns

1. **First-transaction cashback** — onboarding reward after a customer's first wallet payment.  
2. **Merchant co-funded cashback** — merchant pays part of the reward to drive footfall.  
3. **Top-up bonus** — extra value on airtime or wallet top-ups above a threshold.  
4. **Tiered loyalty points** — points per spend unit, with tiers unlocking higher earn rates.  
5. **Bill payment streaks** — rewards for paying utilities through the wallet several months in a row.

Platforms such as EVD System / MoboGage combine wallets, eTopup, and electronic voucher distribution, so rewards can be issued as wallet credit, bonus airtime, or a digital voucher from the same stack.

---

## Implementation Considerations

- **Separate reward balance** — keep rewards distinguishable from customer-funded money for regulatory and refund reasons  
- **Idempotent event processing** — replays must never pay twice  
- **Atomic budget decrement** — prevents overspend under concurrent load  
- **Abuse controls** — device, KYC, and velocity checks against self-dealing and account farming  
- **Transparent terms** — show customers pending vs spendable rewards and expiry dates  
- **Funder statements** — merchants and brands need verifiable cost reports  

---

## FAQ

**What is digital wallet cashback?**  
A reward credited to a customer's wallet after a qualifying transaction, funded by the operator, a merchant, or a brand partner, and governed by campaign rules and budgets.

**How is cashback different from loyalty points?**  
Cashback is usually monetary value in the wallet. Points are a separate unit that converts to value under program rules. Many e-wallet rewards engines support both.

**Why should cashback stay pending for a while?**  
A maturity window aligned with the refund period means a refunded payment can cancel its reward before the customer spends it, avoiding negative balances.

**How do you stop cashback abuse?**  
Limit rewards per verified identity and device, reward settled transactions only, cap per-user earnings, and monitor circular payments between linked accounts.

**How does this relate to MoboGage / EVD System?**  
EVD System is MoboGage's digital value distribution platform family covering wallets, eTopup, and electronic vouchers. See the [EVD System home](https://evdsystem.com/) and [About MoboGage](https://evdsystem.com/about-mobogage/) pages when evaluating product fit.

---

## Glossary Snippet

| Term | Meaning |
|------|---------|
| **Campaign** | A rule set defining trigger, reward, funder, budget, and dates |
| **Funder** | The party paying for the reward (operator, merchant, brand) |
| **Pending reward** | Earned but not yet spendable reward |
| **Maturity window** | Holding period before a reward becomes spendable |
| **Clawback** | Linked reversal of a reward when the qualifying payment is refunded |
| **Per-user cap** | Maximum reward a single customer can earn in a campaign |

---

## Further Reading / Related Industry Resources

- [EVD System home](https://evdsystem.com/) — platform overview for digital value distribution  
- [About MoboGage](https://evdsystem.com/about-mobogage/) — company and product family context  

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not a substitute for vendor due diligence or regulatory advice. MoboGage and EVD System refer to offerings on evdsystem.com.*
