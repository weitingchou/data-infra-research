# 3-Year Roadmap — Toward the Agentic Data & AI Operating System

> Sequences the vision in [`unified-data-ai-platform-plan.md`](./unified-data-ai-platform-plan.md)
> into three phases. The spine is a simple progression — **Foundation →
> Intelligence → Autonomy** — chosen so each capability lands only after its
> prerequisites exist.
>
> *Numeric targets below are **illustrative** — calibrate them against the Quarter-0
> baselines before committing.*
>
> **Settled constraints (given):** **(1) All-build on open source** — no vendor
> lock-in (company policy); vendors inform concepts only. **(2) Ownership** — the
> **Platform Team (our Big-Data team) owns** the unified platform.

---

## North Star

> **Maximize the share of data requests served correctly, at their target SLA,
> with no human making an engine or infrastructure decision** — while the
> professional-services load per data-owner team trends to zero.

Today that share is ~0% (every engine choice is manual, §1 of the vision). The
roadmap moves it to a strong majority by end of Year 3.

---

## Sequencing principles (why this order)

1. **Abstraction before automation.** You cannot auto-route until the **semantic
   layer** decouples logical intent from physical engines. Semantic layer is
   therefore a Year-1 foundation, not a later nicety.
2. **Instrument before intelligence.** The **RAG knowledge base** needs a
   corpus. We capture every query + intent + plan + outcome from Day 1, long
   before agents consume it.
3. **Advisory before autonomous.** Agents earn trust on a ladder: **recommend →
   act-on-approval → act-with-guardrails.** No step is skipped.
4. **Relieve the bottleneck early.** The **engine-choice recommender** is a
   Year-1 quick win that directly attacks the N:1 professional-services pain
   (§2) before the heavier automation arrives.
5. **One source of truth first.** Make Iceberg-on-MinIO canonical and add the
   SSD **cache** tier early; defer bounded native serving copies until routing
   is automated.
6. **Converge where demand is.** Online serving and the ML-platform merge roll
   out against real use cases, not big-bang.

---

## The agentic maturity ladder (the spine of the plan)

| Level | Who decides | Who acts | Example |
|---|---|---|---|
| **L0 — Manual** *(today)* | Human (owner + us) | Human | Owner picks Doris vs Lakehouse; PS prototypes |
| **L1 — Advisory** *(Year 1)* | Agent recommends, **human decides** | Human | Console recommends the engine from declared intent |
| **L2 — Assisted** *(Year 2)* | Agent decides, **human approves** | Agent (on approval) | Auto-routing/tiering applied after a one-click OK; Trino fallback |
| **L3 — Autonomous + guardrails** *(Year 3)* | **Agent decides & acts**, human on-loop | Agent | Auto-scale, auto-recover, auto-route within policy; humans set policy & handle exceptions |

Every agentic capability (routing, ILM tiering, query optimization, ops,
scaling, stewardship) walks this same L0→L3 ladder on the timeline below.

---

## At a glance

| Workstream (→ vision pillar) | Year 1 — Foundation | Year 2 — Intelligence | Year 3 — Autonomy |
|---|---|---|---|
| **Semantic layer** (P2) | Build; certify top domains | Drives compilation & routing | Universal; all access through it |
| **Unified storage / ILM** (P1) | Iceberg = SoT; SSD cache (Level B) | Auto-tier A/B/C (assisted) | Autonomous tiering |
| **Telemetry → RAG KB** (self-reinforcing) | Capture 100% of queries+intent | KB live; grounds decisions | Closed-loop, compounding |
| **Intent & routing** (P1/§5) | Intent API v1 (rule-based) + recommender | Auto-route (assisted) + dialect compilation | Fully intent-driven, autonomous |
| **Agentic control plane** (P4) | Safe-exec foundation (test/preview/domains) | Assisted ops agents | Autonomous ops, scaling, stewardship |
| **Unified Data + AI** (P3) | Plan ML convergence; serve first training data | Workspace v1; Zero-ETL off Oracle | ML platform merged; one workspace |
| **Online serving** (§6) | Harden streaming backbone | Feature store + product-facing OLAP (first cases) | Product-facing serving GA |
| **Org / professional services** | Recommender deflects routine asks | PS shifts to complex cases + curation | PS repurposed to platform/agent curation |
| **End-user access** (§3 paradigm) | Hand-built reports / dashboards (analysts) | + conversational Q&A for operators (first domains) | Growing-share NL self-serve; experts still hand-build complex / high-stakes |

---

## Quarter 0 — Baseline & foundational decisions *(pre-phase, ~1 quarter)*

Short runway to make the decisions everything else depends on.

- **Baseline the metrics** that define success: current query p95/p99 latency &
  cost, professional-services ticket volume per owner team, dataset-onboarding
  lead time, manual-engine-choice rate (≈100%).
- **Build-vs-buy is settled — ALL BUILD on open source** (company policy: no
  vendor lock-in). Quarter 0 instead selects the **OSS components** to assemble on
  the existing Iceberg / Trino / Doris / Spark / Kafka base — semantic layer,
  feature store, KB / vector store, agent framework. Managed products
  (Databricks, Lakebase, …) are reference architectures only.
- **Choose the semantic-layer technology** and how it integrates with Trino + Doris.
- **Decide the real-time OLAP path** (Doris vs. + StarRocks / ClickHouse / Pinot).
- **Stand up telemetry capture** so the RAG corpus starts filling immediately.

**Gate to Year 1:** baselines published; **OSS components selected** (build-vs-buy already settled); semantic-layer tech chosen; telemetry flowing.

---

## Year 1 — FOUNDATION
**Theme: lay the decoupling + data foundation, and relieve the engine-choice bottleneck.**

The goal of Year 1 is *not* agents — it is the substrate that makes agents
possible, plus one high-ROI quick win.

**What we achieve this year:**
- **Decouple intent from physical engines.** Stand up the semantic layer (v1) and
  certify it for the highest-volume domains, so users and tools query one logical
  model instead of raw Doris/Trino tables. This is the foundation everything else
  builds on — you can't auto-route a request until intent is expressed
  independently of the engine.
- **Make Iceberg the single source of truth.** Land all new datasets in
  Iceberg-on-MinIO as the canonical record, and add a local SSD cache to recover
  query latency without creating a second copy or format. One governed source
  keeps every serving tier reconcilable and ends the lake-vs-warehouse split.
- **Instrument everything from Day 1.** Capture 100% of queries, declared intent,
  query plans, cost, and SLA outcomes into a query-history store. We start now —
  well before any agent uses it — because the knowledge base that later grounds
  routing and optimization needs a large corpus that only accumulates over time.
- **Relieve the engine-choice bottleneck with a quick win.** Ship an advisory
  recommender that suggests Doris vs. Lakehouse from each user's declared
  requirement, so owners stop guessing and we stop being paged for routine
  choices. It delivers value early and builds trust before heavier automation lands.

| # | Initiative | Ships | Pillar |
|---|---|---|---|
| 1.1 | **Semantic layer v1** | Central metric/definition catalog, lineage, governance constraints; certify the top domains by query volume | P2 |
| 1.2 | **Iceberg as single source of truth** | Policy + migration: all new datasets land in Iceberg-on-MinIO; "SoT ≠ single copy" doctrine adopted | P1 |
| 1.3 | **SSD cache tier (ILM Level B)** | Local-NVMe cache for Trino/Doris external Iceberg tables — recovers latency without a second format | P1 |
| 1.4 | **Telemetry pipeline** | Capture 100% of queries + declared intent + plans + cost + SLA outcome into a query-history store (the RAG corpus seed) | KB |
| 1.5 | **Engine-choice recommender v1 (advisory, L1)** | Console suggests Doris vs Lakehouse from the user's declared requirement — the first dent in the N:1 bottleneck | P2/§2 |
| 1.6 | **Intent API v1** | Users declare WHAT + SLA; a **rule-based** mapper picks the engine (precursor to agentic routing) | P1/§5 |
| 1.7 | **Safe-execution foundation** | Lightweight cloud testing, preview environments, domain boundaries — the prerequisite for any future autonomy | P4 |

**Persona impact:** Data Owners get *recommendations* instead of guessing or
waiting on us; Data Users get consistent, certified metrics; the Platform Team
stops being paged for every routine engine choice.

**Phase gate → Year 2:**
- Semantic layer covers the datasets behind the majority of query volume.
- 100% of queries/intent captured in telemetry.
- Recommender deflects a meaningful share of professional-services engagements.
- SSD cache live; Iceberg = SoT for all new datasets.

---

## Year 2 — INTELLIGENCE
**Theme: turn the corpus into a RAG knowledge base; agents go advisory → assisted; automate tiering; light up online serving.**

**What we achieve this year:**
- **Move agents from advisory to assisted (L2).** Auto-routing and ILM tiering now
  act on a one-click approval (with a Trino-on-Iceberg fallback for safety), so
  manual engine choice disappears for most workloads. Agents act but a human still
  approves — earning trust before we hand over full autonomy.
- **Turn the corpus into a live knowledge base.** Stand up retrieval (RAG) over the
  query/intent/outcome history so agents can ground routing and optimization in
  what has actually worked here before. This is what makes the assisted decisions
  better than generic heuristics.
- **Begin online serving and ML convergence.** Light up the first product-facing
  real-time cases on the streaming backbone + feature store, and serve the first
  model-training data from the platform instead of legacy Oracle. We roll these
  out against real use cases rather than big-bang.
- **Give business operators conversational Q&A.** In the first well-governed
  domains, operators ask questions in natural language and an agent answers
  through the semantic layer so the result is correct, not just fluent. Analysts
  keep hand-building everything more complex.

| # | Initiative | Ships | Pillar |
|---|---|---|---|
| 2.1 | **RAG Knowledge Base (live)** | Retrieval over the query/intent/outcome corpus; first agent use cases ground decisions in it | KB |
| 2.2 | **Auto-routing + ILM tiering (assisted, L2)** | ILM Levels A/B/C automated; routing decision agent acts on approval, with **Trino-on-Iceberg always-correct fallback** | P1/§5 |
| 2.3 | **Dialect-safe query compilation** | Semantic-layer compiles to native Doris **or** Trino SQL; SQLGlot/Calcite transpiler for raw SQL | §5 |
| 2.4 | **Query-optimization agent (RAG-grounded)** | Suggests/auto-applies plans from past winning executions | P4/KB |
| 2.5 | **Online serving tier — first cases** | Mature Kafka→Flink backbone; online/feature store (Cassandra/Lakebase) for ML **inference**; real-time OLAP for product-facing analytics | §6 |
| 2.6 | **Unified Data+AI workspace v1** | Integrate ML workflows; **serve training data from the platform** (Zero-ETL off Oracle) for first model teams | P3 |
| 2.7 | **Assisted ops agents** | Auto-tune + load-testing agents recommend/act-on-approval, grounded by the KB | P4 |
| 2.8 | **Conversational analytics (first domains)** | Business operators ask in natural language; an agent answers **through the semantic layer** (for correctness) on well-governed domains — analysts keep hand-building the rest | P2/§3 |

**Persona impact:** manual engine choice disappears for most workloads; Data
Scientists train on platform-served data (not legacy Oracle); first **Product /
App** teams consume online serving; professional services shifts from routine
picks to genuinely complex cases.

**Phase gate → Year 3:**
- KB grounds a substantial share of routing/optimization decisions.
- Auto-tiering in production; manual engine choice eliminated for the majority of workloads.
- First product-facing serving use cases live against an SLA.
- Training data for first models served by the platform.
- Safe-exec maturity sufficient to permit limited autonomy.

---

## Year 3 — AUTONOMY
**Theme: agents operate the platform within guardrails; fully intent-driven; closed-loop knowledge base; Data + AI converged.**

**What we achieve this year:**
- **Let agents run the platform within guardrails (L3).** Routing, tiering,
  scaling, recovery, and stewardship run autonomously, with humans on-loop to set
  policy and approve only high-risk actions. The platform shifts from tools we
  operate to a control plane that operates itself.
- **Reach a fully intent-driven platform.** Users and agents declare what they
  need and the platform decides the engine, sizing, and tiering on its own — no
  human in the engine or infrastructure decision, which is the North Star. This is
  where the manual-choice bottleneck from §1 finally closes.
- **Run one converged Data + AI platform.** The separate ML platform is merged into
  a single workspace where the same agents orchestrate both training and inference
  over the same governed data — one platform to operate and govern, not two.
- **Generalize product-facing serving to GA.** Online serving runs at 99.9%+ SLA
  with hard workload isolation, and operator-facing conversational self-serve
  covers a growing share of routine questions — while experts still hand-build the
  complex, high-stakes work.

| # | Initiative | Ships | Pillar |
|---|---|---|---|
| 3.1 | **Agentic control plane (autonomous, L3)** | Auto-tune, auto-scale, auto-recover, autonomous stewardship — human **on**-loop, approval only for high-risk actions | P4 |
| 3.2 | **Fully intent-driven platform** | Users/agents declare intent; platform autonomously routes, sizes, tiers, prepares — engine choice fully automated | P1/§5 |
| 3.3 | **Closed-loop RAG knowledge base** | Agents continuously write outcomes + retrieve; decision quality measurably compounds with usage | KB |
| 3.4 | **Unified Data + AI platform (complete)** | ML platform merged into one workspace; agents orchestrate training **and** inference | P3 |
| 3.5 | **Product-facing serving GA** | Online serving generalized with hard workload isolation + 99.9%+ SLAs | §6 |
| 3.6 | **Autonomous infra scaling** | Native continuous load testing + predictive pre-scaling from historical patterns | P4 |
| 3.7 | **Operator-facing conversational AI (broad)** | NL Q&A across governed domains for a **growing share** of routine questions; analysts / DS still hand-build the complex & high-stakes reports, dashboards & models | P2/§3 |

**Persona impact:** Data Owners self-serve end-to-end via intent; the
professional-services team is **repurposed to platform/agent curation** rather
than per-request help; the org runs **one** Data & AI platform. Business
operators increasingly self-serve answers via **conversational AI** (a growing
share), while analysts / data scientists keep hand-building the complex,
high-stakes work and shift toward curation & agent oversight.

**Phase gate (vision realized):**
- Strong majority of requests intent-driven and auto-routed (North Star).
- Majority of ops actions autonomous (human-on-loop); MTTR materially down.
- Unified platform serves all scenarios incl. real-time inference & product serving.
- Knowledge-base metrics show decision quality improving period-over-period.

---

## Cross-cutting tracks (run all three years)

- **Governance, privacy & compliance** — encoded in the semantic layer (Y1),
  enforced by steward agents (Y2→Y3). Non-negotiable gate on every autonomy step.
- **Agent trust & safety** — the guardrail/approval model matures with the
  maturity ladder; every L-step requires an evals + rollback story.
- **Security** — isolation between product-serving (revenue path) and ad-hoc
  analytics throughout.
- **Cost management** — tiering and auto-scaling are also cost levers; track $/query.
- **Change management & enablement** — migrate owners/users onto intent + semantic
  layer; retrain the professional-services team toward curation.
- **Org convergence** — **we (the Platform Team) own the unified platform**; the
  ML Platform's capabilities fold into us — align in Y2, merge ownership by Y3.

---

## Team Plan — Building the Platform

> **All-build raises the bar.** Because we assemble everything on open source
> (no vendor to outsource to), we must staff to **build**, not just integrate.
> Headcounts below are **illustrative** — calibrate to budget and scale. These
> are *platform builders*, distinct from the thousands of analysts / data
> scientists who **use** the platform.

### Operating model
- **Squad-based** — cross-functional pods, each owning a workstream end-to-end.
- **Repurpose + hire** — the existing Big-Data team and the folding-in ML Platform
  team form the core; the scarce new talent is **AI / agent engineering**.
- **Recruit the hard roles in Quarter 0** — LLM/agent engineers and SREs have the
  longest lead time.

### Talent we need (skill mix)
| Talent | Core skills | Builds | Sourcing |
|---|---|---|---|
| **Distributed data-infra engineers** | Trino, Doris, Iceberg, Spark internals, JVM/perf | Storage, query, ILM tiering/cache, CDC | Repurpose Big-Data team + senior hires |
| **Streaming engineers** | Kafka, Flink, exactly-once, low latency | Ingestion templates, streaming backbone, online serving | Repurpose + hire |
| **LLM / agent engineers** ⭐ | Agent frameworks, RAG, tool-use, evals, guardrails | Agentic control plane, RAG KB, conversational analytics | **Scarce — top hiring priority** |
| **ML platform / MLOps engineers** | Feature stores, training/serving, orchestration | Unified workspace, feature store, model serving | ML Platform team folds in + hire |
| **SRE / reliability engineers** | k8s, observability, load testing, capacity, cost | Safe-exec foundation, scaling, product-serving SLAs | Hire — critical for autonomy & §6 |
| **Data governance / security engineers** | Lineage, access control, privacy, compliance | Semantic governance, steward agents | Hire / upskill; pair with agent eng |
| **Full-stack / frontend engineers** | Web, UX | Self-service console, intent API UI, conversational UX | Hire |
| **Applied data scientists (platform)** | Learned optimization, agent eval, ranking | Agent/RAG quality, learned routing/opt, KPIs | Small group; some from internal DS pool |
| **Product managers (platform)** | Platform-as-product, dev-as-customer | Intent UX, prioritization, data-owner enablement | Hire / assign |
| **Eng leadership / architect** | Distributed-systems architecture, org design | Overall design, squad leadership, agent safety | You + principal architect + EMs |

⭐ = the defining new investment for the agentic era, and the hardest to hire.

### Squads, what they build, and the headcount ramp *(illustrative; team size at end of each year)*
| Squad | Builds (→ pillar / §) | Y1 | Y2 | Y3 |
|---|---|---|---|---|
| **Storage & Lakehouse** | Iceberg/MinIO SoT, ILM A/B/C, Trino/Doris, CDC (P1) | 6 | 7 | 7 |
| **Semantic Layer & Governance** | Semantic layer, catalog, lineage, dialect compilation, governance (P2) | 5 | 7 | 7 |
| **Ingestion & Streaming** | Spark/Airflow templates, Kafka/Flink, console ingestion (current + §6) | 5 | 6 | 6 |
| **Agentic / AI Platform** ⭐ | Control plane, RAG KB, LLM agents (route/opt/ops/steward), conversational AI (P4 + KB + §3) | 4 | 9 | 12 |
| **ML Platform** | Unified workspace, online/offline feature store, training & serving (P3) | 3 | 8 | 9 |
| **Online Serving / Real-time** | Real-time OLAP, online store, product-facing serving + isolation/SLA (§6) | 2 | 6 | 7 |
| **SRE / Reliability & Cost** | Safe-exec, load testing, autoscaling, on-call, cost (P4 ops) | 4 | 6 | 8 |
| **DevEx / Enablement & Curation** | Console UX, docs, onboarding, data-owner enablement, PS transition (org) | 3 | 5 | 6 |
| **Leadership / PM / Architecture** | Head, architect, EMs, PMs, agent-safety & governance leads | 4 | 6 | 7 |
| **Total (platform builders)** | | **~36** | **~60** | **~69** |

### Where the people come from
- **Existing Big-Data team →** Storage, Semantic, Ingestion squads (upskill on the
  semantic layer + agentic tooling).
- **ML Platform team →** folds into the **ML Platform squad** (align Y2, merge Y3).
- **Professional-services team →** becomes **DevEx / Enablement & Curation** —
  their engine-selection judgment also **seeds the RAG knowledge base**.
- **Net-new hires (priority order):** LLM/agent engineers → SREs →
  frontend/conversational → applied-AI data scientists → governance engineers.

### Phase emphasis
- **Year 1** — weight on Storage / Semantic / Ingestion / DevEx; **seed** the AI
  squad (telemetry + KB design + recommender).
- **Year 2** — the big **AI/agent + ML-platform + online-serving** ramp.
- **Year 3** — growth slows; shift from *build* to *operate*; reliability &
  agent-safety scale with autonomy.

### Critical leadership roles to name early
- **Head of Platform** *(you)* — accountable owner of the unified platform.
- **Principal Architect** — coherence across pillars (the single technical spine).
- **Agent Safety & Trust owner** — owns the guardrail / approval model + evals
  along the L0→L3 maturity ladder. *Non-optional for autonomy.*
- **Data Governance lead** — privacy / compliance encoded in the semantic layer.

---

## Open questions → when they get decided

| Vision §9 open question | Decided by |
|---|---|
| Build vs. buy (platform) | **Settled — all-build on OSS** (no vendor lock-in) |
| Semantic-layer technology | **Quarter 0 → Year 1** |
| Real-time OLAP path (Doris vs StarRocks/ClickHouse/Pinot) | **Quarter 0** |
| Zero-ETL scope & order (Oracle first?) | **Year 1 → Year 2** |
| Agent trust / guardrail model | **Year 2 (and hardened in Year 3)** |
| Online-serving engine & feature-store backend | **Year 2** |
| Data-owner self-service enablement | **Year 1 (recommender) → Year 3 (intent)** |
| Professional-services → automation transition | **Year 2 → Year 3** |
| Ownership & org convergence | **Settled — *we* own the platform**; ML Platform folds in (Y2 align → Y3 merge) |

---

## KPIs by year *(illustrative — set real targets off Quarter-0 baselines)*

| Metric | Baseline (now) | Year 1 | Year 2 | Year 3 |
|---|---|---|---|---|
| **Intent-driven / auto-routed requests** | ~0% | rule-based for new requests | majority (assisted) | strong majority (autonomous) |
| **Manual engine-choice rate** | ~100% | falling (recommender) | minority | near-zero |
| **PS engagements per owner team** | high | −40–50% | −60–70% | curation only |
| **Telemetry coverage of queries** | partial | 100% | 100% | 100% |
| **Semantic-layer coverage (of query volume)** | ~0% | majority | ~all | ~all |
| **RAG-grounded agent decisions** | 0 | 0 (corpus building) | growing share | dominant |
| **Training data served by *our* platform** | low | first datasets | first model teams | default |
| **Product-facing serving use cases** | 0 | 0 | first cohort | GA |
| **Autonomous ops actions (human-on-loop)** | 0 | 0 | assisted | majority |
| **Routine questions answered conversationally** | 0% | foundation only | first domains | growing share (experts still build complex) |

---

## Key risks & mitigations

| Risk | Mitigation |
|---|---|
| **Agents act wrongly in production** | Maturity ladder (advisory→assisted→autonomous); always-correct Trino fallback; preview envs; evals + rollback per step |
| **Semantic-layer adoption stalls** | Make it the *only* path to certified metrics; recommender + intent API create pull; certify high-value domains first |
| **RAG KB is low-signal early** | Capture rich telemetry from Day 1; start with advisory use cases where wrong suggestions are cheap |
| **Org friction merging ML platform** | Align in Y2 on shared workspace before merging ownership in Y3; lead with shared wins (serving training data) |
| **Cost of running two engines + copies** | ILM keeps one Iceberg copy + cache; bounded native copies only for real-time/high-QPS; track $/query |
| **Professional-services team disruption** | Reframe as curation/escalation, not redundancy; their judgment seeds the KB |
| **All-build = heavier engineering load** | OSS-only is policy, not optional — budget for the build, lean on mature OSS (Iceberg/Trino/Doris/Spark/Kafka/Flink + OSS semantic layer / feature store), and sequence so each phase ships usable value |

---

## TL;DR

- **Year 1 — Foundation:** semantic layer, Iceberg-as-SoT + SSD cache, full
  telemetry capture, and an **advisory** engine-choice recommender that starts
  dissolving the N:1 bottleneck.
- **Year 2 — Intelligence:** the **RAG knowledge base** goes live, agents move to
  **assisted** auto-routing/tiering/optimization, online serving lights up, and
  ML training starts being served by the platform.
- **Year 3 — Autonomy:** an **agentic control plane** runs the platform within
  guardrails, the platform is **fully intent-driven**, the self-reinforcing loop closes, and
  Data + AI converge into one unified platform.
