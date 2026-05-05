# WhatsApp Business CRM with AI

> A multi-tenant SaaS platform integrating WhatsApp Business API with OpenAI for conversational automation, broadcast messaging, and AI-powered customer engagement.

---

## Context

Businesses across India and the Gulf increasingly rely on WhatsApp as their primary customer channel. Existing tools (Wati, AiSensy, Interakt) work, but each enterprise we worked with had specific workflows that off-the-shelf platforms couldn't accommodate — custom approval flows, AI integration into business logic, deep CRM integration.

We built a multi-tenant WhatsApp Business CRM platform that gives clients full operational control: template management, broadcast messaging, group orchestration, and an AI auto-reply layer for first-touch customer interactions.

---

## My Role

**Head of Engineering & Delivery (CTO)** — Vriksha Techno Solutions

- Architected the multi-tenant data model and tenant isolation strategy
- Owned the WhatsApp Business API integration design, including the Meta template approval workflow
- Made the call to integrate OpenAI directly via API (not through LangChain or other frameworks)
- Led the engineering team through prompt engineering, eval design, and production deployment of the AI layer
- Defined cost-monitoring and rate-limiting strategies for AI usage

---

## Architecture (high level)

````````mermaid ````
flowchart TD
    A[Client Tenant Dashboard] --> B[NestJS API Gateway]
    B --> C[Auth + Multi-Tenant Resolver]
    C --> D[Template Manager]
    C --> E[Broadcast Service]
    C --> F[Conversation Engine]

    D --> M[Meta Template Approval API]
    E --> W[WhatsApp Business Cloud API]
    F --> W

    F --> G[AI Reply Service]
    G --> H[OpenAI API]
    G --> I[Conversation Context Store]

    F --> J[Webhook Receiver]
    J --> W

    K[Mobile App - Customer-facing] --> B
    L[Web App - Operator] --> B

    style G fill:#1f3864,color:#fff
    style H fill:#1f3864,color:#fff
```

---

## Tech Stack

- **Backend:** NestJS, TypeScript, Node.js
- **Database:** MySQL (multi-tenant data isolation), Redis (session + rate limiting)
- **AI:** OpenAI API (direct integration, no framework abstractions)
- **Messaging:** WhatsApp Business Cloud API (Meta), webhook-driven
- **Frontend:** React (operator dashboard), Flutter (mobile companion)
- **Infrastructure:** AWS (EC2, S3 for media, CloudWatch for monitoring)

---

## Key Technical Decisions

**Direct OpenAI API over LangChain.**
We evaluated LangChain early. After two prototypes, we chose direct API calls. Reasons: simpler debugging, predictable cost, no framework upgrade churn, easier handover to engineers without ML background. The "agent framework" abstraction obscured what was actually happening on the wire.

**Webhook-driven conversation engine, not polling.**
WhatsApp's pricing model and quality ratings reward fast acknowledgment. We built a dedicated webhook receiver that decouples message ingestion from processing — bursty incoming traffic doesn't block AI generation or business logic.

**Meta template approval as a first-class workflow, not an afterthought.**
Most teams underestimate this. Templates take 24–48 hours to approve, can be rejected for arbitrary reasons, and need careful versioning. We built a proper approval state machine and a "template warm-up" pattern — staging templates well before clients need them.

**Eval-driven prompt engineering, not vibe-driven.**
We built a small eval set per use case before picking a model. Output quality varied dramatically by use case — one client's lead-qualification flow scored higher with GPT-4o, another's appointment-booking flow performed better with a smaller model. Without evals, we'd have over-paid for compute.

**Strict separation between AI and deterministic code.**
The AI layer suggests responses and qualifies leads. Anything that touches money, schedules, or customer records is handled by deterministic code. The AI is a feature, not the system of record.

---

## Scale & Outcomes

- Multi-tenant deployment across multiple business clients in India and the Gulf
- Template approval pipeline reduced rejection rate by introducing structured pre-checks
- AI auto-reply handled first-touch responses before routing complex queries to human operators
- Cost-per-conversation kept predictable through prompt optimization and tiered model routing

(Specific tenant counts, message volumes, and revenue figures are confidential.)

---

## What I'd Do Differently

- **Build the eval framework on day one.** We bolted it on at month two. If we'd had it from the start, we'd have shipped the first version with a better model selection and saved ~30% in AI costs.
- **Invest earlier in cost observability.** OpenAI bills can move quietly. A dashboard for token usage per tenant per use case should have existed before we shipped to production, not after we got the first surprise invoice.
- **Decouple the prompt store from the codebase.** We initially shipped prompts as code constants. Later we moved them to a versioned datastore so non-engineers (and the AI itself, with guardrails) could iterate on prompts without redeploys.

---

## Note

This is a sanitized case study of production work delivered to private clients. Source code is not included due to confidentiality. Architectural decisions, technology choices, and lessons described here are original and reflect my direct engineering leadership on the project.
