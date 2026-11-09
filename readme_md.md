# Amazon AI Seller Operations Assistant (MCP Engine)

An enterprise-grade, autonomous orchestration engine built on Anthropic's **Model Context Protocol (MCP)**. This system automates high-frequency Amazon Seller Central operations—spanning ad optimization, inventory velocity tracking, and order analytics—while enforcing strict transactional safety via a **two-phase, human-in-the-loop approval pipeline**.

[![MCP Standard](https://img.shields.io/badge/MCP-Streamable%20HTTP-blue?style=flat-square)](https://modelcontextprotocol.io/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![Database](https://img.shields.io/badge/Storage-Supabase%20%2F%20Postgres-3ECF8E?style=flat-square&logo=supabase)](https://supabase.com/)
[![Amazon APIs](https://img.shields.io/badge/API-SP--API%20%26%20Ads-FF9900?style=flat-square&logo=amazon)](https://developer-docs.amazon.com/sp-api/)

---

## ⚡ The Problem: Unchecked LLMs on Live Marketplace APIs

Granting standard LLM agents direct, unconstrained write access to marketplace APIs like Amazon Selling Partner API (SP-API) or Amazon Advertising API presents critical operational and business risks:
* **Catastrophic Financial Hallucinations:** Erroneous ad bid spikes, mass automated repricing glitches, or unintentional inventory suppression.
* **Rate-Limit Choking:** Naive autonomous loops rapidly exhaust Amazon's strict token-bucket rate limits (0.5 to 5 req/sec per endpoint).
* **Compliance & Audit Blindspots:** Lack of deterministic logging on automated changes risks seller account health and policy suspensions.

## 🛠️ The Architecture: Decoupled Detection & Controlled Mutation

This repository implements a modular, streamable HTTP MCP server that treats safety as an infrastructure pattern, rather than a prompt engineering instruction.

```
┌─────────────────┐       Streamable HTTP       ┌─────────────────────────────────┐
│                 │ ◄─────────────────────────► │                                 │
│  Anthropic LLM  │                             │      TypeScript MCP Server      │
│ (Claude / Tool) │                             │   (Orchestration & Validation)  │
│                 │                             │                                 │
└─────────────────┘                             └───────────────┬─────────────────┘
                                                                │
                                        Approval-Gated Pipeline │
                                                                ▼
┌─────────────────────────────────┐   Rate-Buffer /    ┌─────────────────────────────────┐
│                                 │   Cache Master     │                                 │
│     Amazon SP-API & Ads API     │ ◄────────────────► │   Supabase / Postgres Master    │
│  (Mutations only post-approval) │                    │    (Audit Logs, Caching, ACL)   │
│                                 │                    │                                 │
└─────────────────────────────────┘                    └─────────────────────────────────┘
```

### Core Capabilities

* **Two-Phase Approval-Gated Write Pipeline:** 
  Separates autonomous detection (Phase 1: Read/Analyze/Propose) from execution (Phase 2: Approval/Commit). The LLM can independently analyze sales velocity or detect bid inefficiencies, but write operations generate a cryptographically verifiable proposal stored in the Postgres audit log awaiting manual sign-off.
* **Rate-Limit Resilience via Postgres Master Cache:** 
  Acts as an intelligent buffer between autonomous workloads and Amazon's REST endpoints, drastically slashing redundant API roundtrips and absorbing spiky traffic bursts without triggering `429 Too Many Requests`.
* **Configurable Autonomy Thresholds:** 
  Adjust organizational risk tolerance per workflow (e.g., enable full autonomy for non-revenue reports, require one-click human verification for bid changes above $X.XX, and enforce dual-stakeholder approval on inventory deletions)—all via configuration without rebuilding the codebase.
* **Streamable HTTP MCP Transport:** 
  Native implementation of Anthropic's Model Context Protocol over streamable HTTP, enabling low-latency tool calling, transparent context feeding, and seamless connection to external multi-agent systems.

---

## 🚀 Key Modules

| Domain | Automated Analysis & Detection | Guarded Action / Human-in-the-Loop Gate |
| :--- | :--- | :--- |
| **Sales Analytics** | Real-time margin calculation, buy-box suppression monitoring, anomaly detection. | Automated report distribution; incident alert escalation. |
| **Ads Management** | Negative keyword harvesting, ACoS threshold deviation detection. | Bulk keyword bid adjustments, campaign budget allocations. |
| **Order Operations** | Order velocity tracking, FBA vs. MFN split analytics, return spikes. | Batch cancellation approvals, customer service ticket routing. |
| **Inventory Planning** | Days of Supply (DoS) depletion forecasts, stockout projections. | Restock shipment creation, stranded inventory disposition. |

---

## 🚦 Getting Started

### Prerequisites

* Node.js `>= 20.x`
* Python `>= 3.11` (for optional analytical sub-routines)
* Supabase instance (Local via CLI or Cloud)
* Amazon SP-API & Amazon Ads Developer Credentials

### Quick Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/styzzz/amazon-ai-module-seller-assistant.git
   cd amazon-ai-module-seller-assistant
   ```

2. **Configure Environment Variables:**
   ```bash
   cp .env.example .env
   # Add your SP_API_LWA_APP_ID, ADS_CLIENT_ID, SUPABASE_SERVICE_ROLE_KEY, etc.
   ```

3. **Install Dependencies & Run Migrations:**
   ```bash
   npm install
   npx supabase db push
   ```

4. **Start the MCP Server:**
   ```bash
   npm run dev
   ```

---

## 🔒 Security & Compliance

* All Amazon credentials and refresh tokens are encrypted at rest.
* No raw tokens or human-identifiable seller data are persisted in prompt logs.
* Direct writes to Amazon endpoints require an explicit HMAC-signed approval payload.