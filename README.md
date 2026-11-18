# Aegis Automation

Modular AI agents for Amazon seller operations.  
**AI proposes. You decide.**

Five modules with Claude tool use. Write actions are never executed by the model — they queue for human approval.

| Module | Read tools | Write tools (propose only) |
|---|---|---|
| Sales | `sales_summary`, `top_products`, `product_margin` | `propose_price_change` |
| Order Operations | `list_orders`, `late_shipment_risk`, `returns_report` | `propose_refund` |
| Marketing & Growth | `campaign_performance`, `ad_efficiency_flags` | `propose_ad_budget_change` |
| Inventory & Storage | `inventory_levels`, `replenishment_recommendations`, `storage_risk` | `propose_restock` |
| Administration | `account_health`, `reconcile_payouts`, `compliance_notices` | none |

Write tools never execute. They queue a pending action; a human approves it outside the model (`ActionQueue.approve`). The assistant has no approve tool.

## Status
- Runs on **synthetic fixture data** (`seller_agent/backend.py`). No Amazon SP-API, Ads API or Shopify backend is implemented.
- No audit log, approval thresholds, injection screening or monitors yet.

## Run
```
pip install -e ".[dev]"
pytest
export ANTHROPIC_API_KEY=...   # never commit
python -m seller_agent         # /pending, /approve A0001, /reject A0001
```

## About
Aegis Automation is a control-first system for Amazon sellers. AI analyzes and proposes. You approve.
