# Logos Tax Systems — Platform System Map
_Last updated: 2026-06-23_

Comprehensive Mermaid map of **all repos**, **features/modules**, and **data flows**. This is the successor to the original monolithic `taxpert-architecture.md` diagram.

For product pillars and journeys see [Product Architecture](./product-architecture.md). For focused deployment/route/AI diagrams see [Technical Architecture](./technical-architecture.md).

---

## Full System Map

```mermaid
graph TB
    subgraph USERS [Users]
        U1[BusinessOwner]
        U2[CPA / TaxPro]
        U3[PlatformAdmin]
        U4[AnonymousVisitor]
    end

    subgraph FE [Front-End Layer]
        subgraph WEB [TT-website — Next.js 16 · :3002]
            WEB_HOME[/ — marketing homepage]
            WEB_PROD[/product /for-cpas /pricing /faq /about]
            WEB_LEGAL[/legal/privacy /legal/eula]
            WEB_WAIT[/api/waitlist — Brevo]
            WEB_CHAT[/api/chat — pre-sales]
            WEB_KNOW[src/data/product-knowledge.ts]
        end

        subgraph TT [taxpert-therapy — Next.js 14 · :3003]
            TT_AUTH[Auth and Onboarding\n/login /signup /onboarding/step1-3\n/auth-callback /password-reset]
            TT_PILLARS[Five Pillars\n/citadel /enchiridion\n/ledger /vault]
            TT_DASH[Dashboard\n/dashboard — journey home\nConversationFirstDashboard]
            TT_LOGOS_PAGE[Logos full page\n/logos /logos/history\n/logos/session/id]
            TT_WIDGET[LogosWidget\nfloating chat in layout.tsx\nhidden on /strategies /logos /onboarding]
            TT_STRAT[Strategy agent\n/strategies /strategies/id]
            TT_ENG[CPA Engagement\n/dashboard/my-cpas\n/api/engagement/requests|end]
            TT_SUB[Subscription\n/dashboard/subscription\n/api/stripe/*]
            TT_INT[Integrations UI\n/dashboard/quickbooks\n/dashboard/bank-analysis\n/api/plaid/* /api/qbo/*]
            TT_ADMIN[Admin\n/admin /cpa-admin]
            TT_BFF[BFF API Routes\n/api/tax-advisor/*\n/api/logos/* /api/v1/*\n/api/webhooks/*]
            TT_SDK[lib/apiCoreClientEnhanced.ts\nwraps packages/sdk]

            subgraph LOGOS_LIB [lib/logos — Document Parser]
                DOC_EXTRACT[documentExtractor.ts\nrule-based then Claude Haiku]
                RULE_PARSER[parsers/ruleBasedParser.ts]
                HAIKU_PARSER[parsers/claudeHaikuParser.ts]
                RULES[parsers/rules W-2 1099 Schedules K-1 1040]
                CTX_ASSEM[contextAssembler.ts]
                PII_CLEAN[sanitizePii.ts]
                LOGOS_CTX_SVC[logosContext.service.ts\npatchUserTaxContext]
            end
        end

        subgraph CPA_FE [TT-cpa-platform — Next.js 16 · :3001]
            CPA_LAND[/ — marketing landing]
            CPA_AUTH[Auth /login /signup]
            CPA_DASH[Dashboard\n/dashboard stats requests clients]
            CPA_CRM[FirmClients CRM\n/dashboard/clients/id notes]
            CPA_REQ[Request Queue\n/dashboard/requests accept decline]
            CPA_RET[Tax Return Kanban\n/dashboard/returns 9 columns]
            CPA_EARN[Earnings /dashboard/earnings]
            CPA_MKT[Marketplace\n/marketplace /marketplace/cpaId]
            CPA_PROFILE[Firm Profile /dashboard/profile branding]
            CPA_API[API Routes\n/api/clients /api/requests\n/api/returns /api/handoffs/sync\n/api/marketplace /api/qbo]
            CPA_MOD[src/modules identity clients returns]
            CPA_CORE[lib/tt-core proxy to TT-api-core]
        end
    end

    subgraph BE [Back-End Layer]
        subgraph CORE [TT-api-core — NestJS Monorepo · :8085]
            CORE_AUTH[auth — JWT blacklist guards]
            CORE_ORGS[orgs — multi-tenant]
            CORE_WS[workspaces]
            CORE_CPA[cpa — firm profile]
            CORE_INVITE[invitations]
            CORE_LOGOS[logos — AI conversation RIE ROI]
            CORE_AI[ai — categorize analyze deductions risk]
            CORE_TAXADV[tax-advisor — public IRC widget\nanon Redis 24h transfer 30d]
            CORE_TAX[tax-analytics]
            CORE_STRIPE[billing — Stripe]
            CORE_PLAID[plaid]
            CORE_QBO[qbo — OAuth sync]
            CORE_LLM[llm router — Claude + OpenAI]
            CORE_EB[event-bridge — webhooks pub/sub]
            CORE_REAL[realtime WebSocket]
            CORE_AUDIT[audit]
            CORE_THREADS[threads]
            CORE_RIE[RIE pipeline]
            CORE_NOTIF[notifications]
            CORE_HANDOFF[handoffs — AI to CPA]
            CORE_ENTITL[entitlements]
            CORE_ADMIN[admin]
            CORE_METRICS[metrics quota]
            CORE_HEALTH[health]

            subgraph SHARED [packages]
                PKG_CORE[packages/core types enums plans]
                PKG_SDK[packages/sdk generated client]
            end

            subgraph STRAT_ENG [logos/strategies — TypeScript NOT RAG]
                SE_ALL[Augusta S-Corp QBI Bonus Depreciation\nHiring Children Family Employment\nRetirement Cost Seg RE Pro Health Acct Plan]
                CROSS_INTEL[Cross-Intelligence Service]
                PDF_SVC[PDF Template Service]
            end
        end

        subgraph QBO_SVC [TT-cpa-platform/qbo-service]
            QBO_SYNC[QBOSyncService]
            QBO_WH[WebhookHandler]
            QBO_SCHED[ScheduledJobProcessor]
            QBO_TAX[TaxAnalyticsProcessor]
        end
    end

    subgraph AI [AI Agentic Layer]
        subgraph COPILOT [TT-dev-copilot — ADK + FastAPI]
            AGENT[root_agent gemini-2.5-flash\ntt_dev_copilot gemini-1.5-flash]
            LOGOS_ADV[agents/logos-tax-advisor deployable]

            subgraph RETRIEVERS [Two RAG Indexes]
                CODEBASE_RAG[retrievers.py — Codebase Index\ntt-canon architecture workflows]
                IRC_RAG[irc_retriever.py — IRC Index\nTitle 26 IRS Pubs 17 334 535 587 946]
            end

            subgraph LOGOS_TOOLS [Logos Advisor Tools]
                LOGOS_CTX[logos_context_tools.py]
                STRAT_TOOLS[strategy_engine_tools.py]
                CALC[calculators/augusta_rule.py]
            end

            subgraph DEV_TOOLS [Dev Copilot Tools]
                GH_TOOLS[github_tools.py]
                TEST_TOOLS[test_execution_tools.py]
                DB_TOOLS[database_tools.py]
                SRV_TOOLS[server_management_tools.py]
            end

            ZENO[zeno_bot.py Slack]
        end

        subgraph MCP_DEV [TT-mcp-server — dev only]
            MCP_TOOLS[logos_agent_chat\nTT-api-core proxy]
        end
    end

    subgraph INFRA [Infrastructure and Data]
        SUPA[(Supabase PostgreSQL\nyhrbwyarzrjwgkthojzu)]
        PRISMA[Prisma ORM]
        REDIS[(Redis sessions rate limit pub/sub)]
        STRIPE_EXT[Stripe]
        PLAID_EXT[Plaid API]
        QBO_EXT[QuickBooks Online]
        GCS[(GCS artifacts RAG docs)]
        GCS_IRC[(GCS IRC staging)]
        VERTEX_CB[Vertex AI Codebase Index]
        VERTEX_IRC[Vertex AI IRC Index]
        ANTHROPIC[Anthropic Claude]
        OPENAI[OpenAI]
        SENTRY[Sentry]
        GCP_LOG[Cloud Logging]
        GITHUB[GitHub API]
    end

    %% Users to front-ends
    U4 --> WEB_HOME
    U4 --> WEB_CHAT
    U1 --> TT_DASH
    U1 --> TT_PILLARS
    U1 --> TT_LOGOS_PAGE
    U1 --> TT_WIDGET
    U2 --> CPA_DASH
    U2 --> CPA_CRM
    U3 --> TT_ADMIN
    U3 --> CPA_FE

    %% TT-website
    WEB_CHAT --> CORE_TAXADV
    WEB_WAIT --> STRIPE_EXT

    %% Document parser
    TT_BFF --> DOC_EXTRACT
    DOC_EXTRACT --> RULE_PARSER
    DOC_EXTRACT --> HAIKU_PARSER
    RULE_PARSER --> RULES
    HAIKU_PARSER --> ANTHROPIC
    DOC_EXTRACT --> PII_CLEAN
    DOC_EXTRACT --> CTX_ASSEM
    CTX_ASSEM --> LOGOS_CTX_SVC
    LOGOS_CTX_SVC --> SUPA

    %% Tax advisor anonymous
    TT_BFF --> CORE_TAXADV
    CORE_TAXADV --> REDIS
    CORE_TAXADV --> IRC_RAG

    %% TT app to backend
    TT_AUTH --> CORE_AUTH
    TT_DASH --> CORE_LOGOS
    TT_DASH --> CORE_TAX
    TT_DASH --> CORE_STRIPE
    TT_DASH --> CORE_AI
    TT_LOGOS_PAGE --> CORE_LOGOS
    TT_STRAT --> CORE_LOGOS
    TT_ENG --> SUPA
    TT_BFF --> CORE_AUTH
    TT_BFF --> CORE_ORGS
    TT_SDK --> PKG_SDK

    %% CPA to backend
    CPA_CORE --> CORE_AUTH
    CPA_CORE --> CORE_ORGS
    CPA_CORE --> CORE_CPA
    CPA_API --> QBO_SVC
    CPA_API --> CORE_QBO
    CPA_API --> CORE_HANDOFF

    %% Backend internals
    CORE_LOGOS --> CORE_LLM
    CORE_LOGOS --> CORE_THREADS
    CORE_LOGOS --> CORE_RIE
    CORE_AI --> CORE_LLM
    CORE_RIE --> SE_ALL
    SE_ALL --> CROSS_INTEL
    CROSS_INTEL --> PDF_SVC
    CORE_LLM --> ANTHROPIC
    CORE_LLM --> OPENAI
    CORE_STRIPE --> STRIPE_EXT
    CORE_PLAID --> PLAID_EXT
    CORE_QBO --> QBO_EXT
    CORE_AUTH --> REDIS
    CORE_EB --> REDIS
    CORE_ORGS --> SUPA
    CORE_THREADS --> SUPA
    PRISMA --> SUPA

    %% QBO microservice
    QBO_SYNC --> QBO_EXT
    QBO_WH --> QBO_SYNC
    QBO_SCHED --> QBO_SYNC
    QBO_TAX --> SUPA

    %% AI layer
    AGENT --> CODEBASE_RAG
    AGENT --> IRC_RAG
    LOGOS_ADV --> IRC_RAG
    CODEBASE_RAG --> VERTEX_CB
    IRC_RAG --> VERTEX_IRC
    VERTEX_CB --> GCS
    VERTEX_IRC --> GCS_IRC
    AGENT --> LOGOS_CTX
    AGENT --> STRAT_TOOLS
    STRAT_TOOLS --> CORE_LOGOS
    LOGOS_CTX --> SUPA
    AGENT --> GH_TOOLS
    GH_TOOLS --> GITHUB
    MCP_TOOLS -.-> CORE
    MCP_TOOLS -.-> AGENT

    %% Infra monitoring
    SENTRY -.-> TT_AUTH
    GCP_LOG -.-> COPILOT
```

---

## Repo Quick Reference

| Repo | Port | Domain | Role |
|------|------|--------|------|
| TT-website | 3002 | logostaxsystems.com | Marketing, waitlist, pre-sales chat |
| taxpert-therapy | 3003 | app.logostaxsystems.com | BO app — five pillars, Logos, engagement |
| TT-cpa-platform | 3001 | cpa.logostaxsystems.com | CPA CRM, kanban, marketplace |
| TT-api-core | 8085 | Cloud Run | NestJS — all business logic |
| TT-dev-copilot | — | Cloud Run | ADK agents, dual RAG indexes |
| TT-mcp-server | — | local dev | Cursor MCP bridge |

---

## How to view

- **Browser:** [viewer.html](./viewer.html) → **Full System Map** tab
- **Markdown:** open this file in any Mermaid-capable preview (Obsidian, GitHub, VS Code)
