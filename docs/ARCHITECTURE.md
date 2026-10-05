# Architecture — Digital Wallet Cashback & Rewards

High-level reference for evaluating **digital wallet cashback**, e-wallet rewards engines, and wallet loyalty points. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Settled tx feed  │────►│ Campaign matcher │────►│ Budget + caps   │
│ (pay/topup/bill) │     │ (rules, segments)│     │ ledger          │
└──────────────────┘     └──────────────────┘     └────────┬────────┘
                                                           │
                                                           ▼
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Funder statements│◄────│ Clawback + expiry│◄────│ Reward ledger   │
│ + liability rpt  │     │ engine           │     │ pending/spendable│
└──────────────────┘     └──────────────────┘     └─────────────────┘
```

## Suggested Reward Record Fields

| Field | Meaning |
|-------|---------|
| `reward_id` | Unique ID for the reward posting |
| `source_event_id` | Idempotent link to the qualifying settled transaction |
| `campaign_id` | Campaign whose rules produced the reward |
| `funder_id` | Operator, merchant, or brand paying for it |
| `amount` | Reward value in wallet currency or points |
| `status` | `pending` / `spendable` / `spent` / `clawed_back` / `expired` |
| `matures_at` | When pending becomes spendable |
| `expires_at` | When unused reward lapses |

## Audit Events Worth Keeping

- Campaign created, edited, paused, and budget changes (who and when)  
- Rule evaluation outcome per event, including "not rewarded" reasons  
- Budget decrements and cap hits  
- Maturity, spend, clawback, and expiry transitions  
- Funder statement generation and export  

## Failure Modes to Design For

- Rewarding authorizations that never settle  
- Budget overspend under concurrent events  
- Refund processed without clawing back its reward  
- Rewards mixed into customer-funded balance, complicating refunds and safeguarding  
- Rule edits mid-campaign applied retroactively without versioning  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [About MoboGage](https://evdsystem.com/about-mobogage/).
