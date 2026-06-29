# Plan: The Agentic Data & AI Operating System

> A 3–5 year conceptual vision for converging our Data Infrastructure and ML
> Platform into a single, unified, agent-native Data & AI platform.
>
> *Synthesized from research (Meta, AWS, a16z, Databricks) and a
> brainstorming discussion, extended with AWS's 2025–2026 publications on
> agent-operated systems. This is the vision/concept note; the **sequenced
> 3-year roadmap** derived from it lives in `roadmap-3-year.md`.*

---

## 1. Context & Starting Point

### Where we are today
- **Data Lakehouse** — Spark ingestion, Iceberg table format, Trino query engine
- **Data Warehouse** — Doris
- **Legacy stack** — Hadoop suite (HDFS, MapReduce, Spark, HBase)
- **Cassandra** — operational, not for analytics

### What we already provide (self-service today)
- **Self-service ingestion console.** Users build and manage ingestion pipelines
  themselves through a **user console** — no tickets to our team. We ship
  **pipeline templates** on **Spark** (batch) and **Spark Streaming** (streaming),
  orchestrated by **Airflow**: Airflow triggers the Spark job for batch and
  supervises the lifecycle of streaming pipelines. Users supply only parameters
  (destination table, CPU / memory to allocate, etc.) and the console **spawns the
  Spark pipeline** for them.
- **Kafka ingestion with configurable routing.** Users push data to **Kafka**,
  then use the console to route it to **Doris** or the **Lakehouse**.
- **Bidirectional CDC replication.** Data can be replicated **between Doris and the
  Lakehouse in either direction via CDC** — so an initial engine choice isn't
  permanent: ingest into one, replicate to the other later.

### Who we serve
- **Data engineering team** (analysts, data scientists) is our primary customer.
- The **ML Platform** is built by a *separate team* today.

### Primary user scenarios
1. **BI charts & reports** — mostly require **real-time** updates to reflect system metrics.
2. **Ad-hoc analysis** — across **long historical time ranges**.
3. **ML model training** — today most training data comes from *legacy* infra
   (e.g., Oracle), **not** served by our team.
4. **Online / product-facing serving (new)** — the product itself reads platform
   data *in the request path* (e.g., Uber-style in-app analytics, real-time ML
   **inference** / personalization), at millisecond latency and high QPS.

> Scenarios 1–3 are **offline** (human-in-the-loop or batch training); scenario 4
> is **online** and is detailed in §6.

### The friction today (what the future must remove)
- **Manual engine choice.** Users must **decide for themselves whether Doris or
  the Lakehouse serves each request**, and manually prepare the data on the chosen
  engine via the console. There is **no auto-tiering or auto-routing** to make that
  decision for them.
- **Knowledge burden on data owners.** Doris and the Lakehouse (Iceberg + Trino)
  have **different performance profiles**, so choosing well means understanding
  **OLAP vs. OLTP** trade-offs. But most **data owners come from an
  application-engineering background** (not analytics / data science), so the
  choice is genuinely hard for them.
- **Professional services as a non-scaling crutch.** To compensate, we run a
  **professional-services team** that helps users pick the right engine and build
  prototypes. It works — but it is **human effort filling the gap left by missing
  automation and abstraction**, and it does not scale — **many data-owner teams,
  one Platform Team**.

### The two core beliefs driving this vision
1. **Adopt Databricks' "unified Data + AI platform" *concept*** — merge the data
   platform and the ML platform into one. (The *concept* only — per company
   policy we **build it ourselves on open source**, not buy the product; see §9.)
2. **The future platform should be agent-native** — LLM agents as a first-class
   orchestration layer, not a bolt-on.

---

## 2. Personas & Ownership Model

Three personas interact with the platform. The boundary is simple: the
**Platform Team** owns *how it runs*, **Data Owners** own *what's in it*, and
**Data Users** own *what we learn from it*. A fourth group — **📱 Product /
Application teams** — enters once we add online serving (§6): they *consume* the
platform's serving endpoints / SDKs in their apps, so they are a **downstream
consumer**, not a core ownership persona. And as agentic AI matures (§3), the
consumer mix gradually shifts: a **growing share** of questions are answered for
**business operators directly via conversational AI** (the **agent itself a
first-class consumer**), while Data Users **keep hand-building** the complex,
high-stakes reports and models and move more of their effort toward semantic /
model curation and agent oversight.

| Persona | Who | Primary responsibility |
|---|---|---|
| 🛠️ **Platform Team** *(us)* | Big-Data team | **Build & operate** the platform and infra components — storage, engines, semantic-layer framework, agent runtimes, control plane. |
| 📦 **Data Owners** | Application / domain teams that **own** the data | **Onboard data** onto the platform and own its meaning, quality, retention & access policy. |
| 🔍 **Data Users** | A *separate* team of data analysts & data scientists | **Consume data** to produce insight, dashboards, and ML models. |

**How they interact today.** Data Users — a *separate* analyst / data-scientist
team — raise the querying requirements; Data Owners (from the *application* team
that owns the data) must pick the platform solution (Doris or Lakehouse) that
meets them. Lacking the OLAP/OLTP background to choose, they escalate to **us**,
or ask professional services to prototype. With **many data-owner teams but a
single Platform Team**, that funnel is the structural bottleneck — so the future
moves the *choice itself* into the platform (intent → auto-routing), letting
owners self-serve and taking us out of the critical path.

**Role codes used below:** **B** = build & operate · **O** = own / feed (onboard
& define) · **U** = use / consume · **—** = minimal involvement.

### Responsibility matrix (persona × component)

| Pillar / Component | 🛠️ Platform Team | 📦 Data Owners | 🔍 Data Users |
|---|---|---|---|
| **P1** · Lakehouse, Iceberg, object storage | **B** | — | — |
| **P1** · Zero-ETL / federation connectors | **B** | **O** onboard & register sources | — |
| **P1** · AI-driven ILM (object lifecycle · SSD cache · bounded serving copy) | **B** | sets retention / SLA inputs | — |
| **P2** · Semantic-layer framework + lineage | **B** | — | — |
| **P2** · Metric & business-logic definitions | provides tooling | **O** author & certify | **U** consume / request |
| **P2** · Privacy & governance constraints | **B** enforce | **O** define per domain | **U** comply |
| **P3** · Workspace, agent framework, compute | **B** | — | **U** primary |
| **P3** · SQL / notebooks / GenAI apps / model building | provides tooling | provide ML-ready data | **O/U** build & own outputs |
| **P4** · Control plane, auto-tuning, load testing | **B** | — | — |
| **P4** · Data steward agents (quality / privacy) | **B** | **O** remediate domain alerts | — |
| **Consumers** · BI, ad-hoc, ML training | serve & SLO | certify source data | **U** consume |
| **Online serving (§6)** · streaming + real-time OLAP + feature store | **B** · serve & SLO | **O** emit events / source data | **U** data scientists define features |

> **Reading the matrix:** the Platform Team builds nearly everything (we are the
> infra builders); Data Owners concentrate in **onboarding (P1)** and
> **definitions / governance (P2)**; Data Users live mainly in the
> **workspace (P3)** and as **consumers** of the semantic layer (P2). **Online
> serving (§6)** adds a downstream consumer — **📱 product / application teams** —
> who call serving endpoints rather than own data.

---

## 3. Vision Statement

**Today: self-service, but only as a toolbox.** Users already stand up ingestion
pipelines, route Kafka streams, and replicate data between Doris and the
Lakehouse from a console — without waiting on us. But the console hands them
low-level controls and still expects the expert decisions:

- They choose Doris or the Lakehouse, size the Spark resources, and prepare the
  data on the right engine — which takes an **OLAP-vs-OLTP** mental model to do well.
- Most data owners are **application engineers, not analysts**, so a
  **professional-services team** stands behind the console to make those calls and
  build prototypes.
- With **many data-owner teams but one Platform Team**, every such decision
  funnels through us — a bottleneck.

**The shift: from controls to intent.** Instead of "ingest to Doris with 8
cores," a user — or an agent acting for them — says **"serve this dataset for a
real-time dashboard at sub-second latency,"** and the platform makes the
implementation decisions:

- **Intent-driven routing.** The platform routes, sizes, tiers, and prepares the
  data itself — choosing Doris, the Lakehouse, a cache, or a bounded serving copy.
- **A global semantic layer** hides the physical engines behind consistent,
  self-describing datasets, so no one needs to tell OLAP from OLTP to get a
  correct, fast answer.
- **An agentic control plane** automates the routing and tiering that are manual
  today, encoding the professional-services team's judgment into the platform so
  it scales.
- **The platform learns from its own usage.** Every query and intent lands here,
  and that usage feeds an **in-house RAG knowledge base** that makes each next
  routing, optimization, and scaling decision better over time — not a static rulebook.

**Convergence: one Data & AI platform.** The data platform and the separate ML
platform merge into a single environment, where the same governed data serves
dashboards, ad-hoc analysis, model training, real-time **inference**, and
product-facing serving alike — with **LLM agents as the central orchestration
layer** across ingestion, storage, analytics, and machine learning *(extends
AWS's lakes + BI + ML vision, ref [8])*.

**The end-user mix shifts too — gradually, never fully.** Today analysts and
data scientists are the main *interface* between the platform and the business,
hand-building reports, dashboards, and models. As agentic AI matures:

- **Business operators can ask in natural language**, and an agent derives the
  answer directly from the platform, grounded in the semantic layer's certified
  metrics so it is *correct*, not just fluent.
- **Analysts and data scientists keep hand-building** the complex, bespoke, and
  high-stakes work; the routine, self-serve questions are what get answered on demand.
- **Personas reshape:** the agent becomes a first-class consumer alongside
  humans, business operators become more direct users, and analysts / data
  scientists shift toward **curating semantics, governing models, and supervising agents**.
- **The plan already supports this:** a platform whose agents declare intent,
  retrieve over a governed semantic layer, and are grounded by the RAG knowledge
  base is exactly what a conversational, operator-facing AI needs to answer correctly.

> **In one line:** move from a *self-service toolbox that still demands
> expertise* to an *intent-driven, agent-operated platform that makes the expert
> decisions for you.*

---

## 4. The Four Pillars

### Pillar 1 — Converged, Zero-ETL Lakehouse Foundation ("Unified Storage")
*Goal: one foundation for both analytical and operational data, at scale.*

- **Converge** the analytical warehouse and the operational data lake onto a
  single foundation built on **open table formats (Iceberg)** + intelligent
  object-storage tiering.
- **Open data foundation for agentic AI** *(AWS, ref [1])*. The foundation must
  be an open, interoperable architecture that lets AI agents **find, access,
  trust, and act on data across systems** — explicitly avoiding vendor lock-in
  via open-source tech, **open table formats, and managed data services**.
  **Iceberg is the linchpin**: one open table format that works across query
  engines and processing frameworks, so engines stay swappable as agents evolve.
- **Reduce brittle pipelines.** Replace manual extract-from-legacy jobs with
  **Zero-ETL integrations** and **federated query** to process data *in place*
  (directly addresses the "training data trapped in Oracle" problem).
- **AI-driven Information Lifecycle Management (ILM).** *Single source of truth
  ≠ single physical copy.* Iceberg-on-MinIO is the canonical record for **all**
  data; derived serving tiers are allowed only if they are auto-derived from
  Iceberg, governed by the same semantic layer, and reconcilable to it. ILM
  operates at **three levels**:
  - **A · Lake object lifecycle** — auto-tier Iceberg files hot→warm→cold and
    compact / expire snapshots. *Format-agnostic — the data stays Iceberg.*
    (cf. AWS S3 Intelligent-Tiering, ref [11]; Databricks Predictive
    Optimization, ref [12].)
  - **B · Local SSD cache tier** — Doris / Trino cache the hot Iceberg working
    set on local NVMe (ephemeral LRU, **not a second copy**), recovering most of
    the latency for ad-hoc and the majority of BI while keeping one format.
    (cf. Databricks disk cache, ref [12]; Doris / StarRocks external-table data
    cache, ref [13].)
  - **C · Serving-copy promotion (bounded exception)** — only the few
    **real-time, high-QPS** dashboards are auto-materialized into **Doris-native**
    on SSD, where Iceberg-on-object-store is genuinely weakest (real-time upserts
    + very high concurrency). Auto-fed from Iceberg; identical semantic logic.
  The **"AI" job of ILM is the promotion / routing decision** — predicting which
  datasets warrant direct-query → cache (B) → native serving copy (C) by latency
  SLA and concurrency, not blindly "moving data into Doris."

  > **Provenance & correction:** the original *"tier between MinIO and Doris"*
  > phrasing came from the **Gemini brainstorm** (ref [10]) — not an AWS/Databricks source
  > (which is why it was uncited). As written it conflated *storage tiering* with
  > *engine / format choice* — promoting data into Doris-native is a **second
  > format**, which is what appeared to break the Iceberg-single-format goal. The
  > three-level model above resolves it and is grounded in real primitives
  > [11]–[13]. **Chosen stance: Hybrid** — Iceberg is the system of record for
  > everything; native serving copies exist only for the few real-time / high-QPS
  > dashboards.

**Maps to our current stack:** Iceberg ✅, Trino (federation) ✅, Doris (hot
OLAP) ✅ — the new work is *Zero-ETL ingestion* and the *AI-driven ILM* (lifecycle · cache · bounded promotion).

**Personas:** 🛠️ *Platform Team* builds the lakehouse, connectors & ILM engine ·
📦 *Data Owners* onboard and register their source datasets · 🔍 *Data Users*
read through engines (no direct ownership here).

---

### Pillar 2 — Global Semantic Layer & "Accessible Analytics"
*Goal: shift data engineering from moving physical tables to curating meaning.*

- The core function of data engineering shifts from **moving data** → **building
  "Accessible Analytics"** *(Meta, ref [7])*: semantically rich, logical datasets instead of raw
  physical tables.
- A **central semantic layer** encodes **meaning, business logic, and privacy
  constraints globally** across the entire data lineage.
- **Outcome:** whether a user loads a real-time metrics dashboard or runs a
  multi-year Trino query, they interact with a **single, consistent,
  self-describing** representation of the business.
- This is also the **prerequisite for trustworthy agents** — agents need a
  semantic layer to reason over the data correctly (and serves the
  "modern BI blueprint" idea from the a16z reference, ref [5]).
- **It is the "trust" leg of agentic data access** *(AWS, ref [1])*. The open
  foundation's promise that agents can *find, access, **trust**, and act* on data
  is only realizable if meaning, lineage, and privacy constraints are encoded
  here — semantics + governance are what make autonomous action safe.
- **Data is the differentiator for gen AI** *(AWS Gen AI white paper, ref [4]).*
  As foundation models commoditize, the model stops being the edge — everyone
  calls the same LLM. The lasting advantage is your **proprietary, well-governed
  data**, so a gen-AI strategy is really a data-platform strategy. Data
  differentiates through four mechanisms:
  - **Grounding / RAG** — your facts, metrics & history retrieved at inference.
  - **Fine-tuning / adaptation** — your domain's vocabulary, patterns, edge cases.
  - **ML features** — a commodity algorithm + years of proprietary signal = the
    real predictive power.
  - **Eval / feedback loops** — your usage data to measure and improve quality.
  *Examples:* a raw LLM cannot answer *"Q3 churn for the enterprise-SMB segment"*
  — only governed data plus the semantic layer's certified metric can ground it;
  and two competitors on the **same** model diverge entirely on who has cleaner,
  PII-governed, well-labelled data. **The point: a generic model on your governed
  data beats a better model on generic data** — and "governed" is exactly what the
  semantic layer (this pillar) provides.

**Personas:** 🛠️ *Platform Team* builds the semantic-layer & governance engine ·
📦 *Data Owners* author metrics, business logic & privacy rules and **certify**
datasets · 🔍 *Data Users* consume certified datasets and request new metrics.

---

### Pillar 3 — Agentic AI Data Science Hub
*Goal: merge data engineering and ML into one workspace; LLMs orchestrate ML.*

- **Unified workspace** where SQL analytics, data processing, and generative-AI
  app development coexist (the Databricks unification, ref [9], made concrete).
- **Hybrid strategy — LLMs orchestrate, they don't replace.**
  - **LLM agents = intelligent architects:** advanced reasoning, strategic
    feature-engineering suggestions, and executable-code generation.
  - **Heavy statistical compute stays distributed:** XGBoost, Spark MLlib over
    structured data on the cluster.
  - **Agents drive the optimizers:** e.g., operate Hyperopt to solve
    cold-start problems and automate model-training recovery.
- This refines the earlier "LLM replaces the data scientist for auto-training"
  idea into a more durable **"LLM as orchestrator of traditional ML"** model.
- **New output modalities** this unlocks: **autonomous AI "data scientists"**
  (LLMs querying governed data directly) and **RAG-native agentic apps** that
  read/write Iceberg tables.

**Personas:** 🛠️ *Platform Team* builds the workspace, agent framework & compute
runtimes · 📦 *Data Owners* keep domain data ML-ready · 🔍 *Data Users* — **this
is their home**: build models, run SQL/GenAI, and direct the agents.

---

### Pillar 4 — Autonomous Operations & Infrastructure Scaling ("Agentic Control Plane")
*Goal: agents run and protect the platform.*

- **From reactive assistants to proactive autonomous systems** *(AWS, Rethinking
  SaaS in the Agentic Era, ref [3]).* The end state is agents that
  **understand, decide, and act with minimal oversight** — the platform shifts
  from tools humans operate to a control plane agents operate, with humans
  setting policy and approving high-risk actions.
- **Auto-optimization:** agents continuously monitor physical execution plans
  and historical query performance to **auto-tune Trino/Spark routing**.
- **An environment where agents can operate effectively *and safely*** *(AWS,
  Architecting for Agentic AI Development, ref [2]).* Combine three things so
  agents can act without breaking production:
  - **Lightweight cloud testing** — cheap, fast, isolated test runs per agent action.
  - **Preview environments** — agents validate changes in disposable replicas
    before they touch production.
  - **Domain-driven structure** — clear domain boundaries so an agent's blast
    radius is scoped and its actions are intelligible.
- **Native, continuous load testing:** distributed, Celery-managed **k6** workers built
  into the deployment pipeline so the platform can absorb **massive ad-hoc
  analytical spikes**.
- **Agents as automated data stewards:** continuously monitor pipelines to
  enforce **privacy, compliance, and data-quality** standards across the
  ecosystem.

**Personas:** 🛠️ *Platform Team* builds & runs the control plane and steward
agents and sets the guardrails · 📦 *Data Owners* act on steward-agent alerts for
their domain · 🔍 *Data Users* benefit from the reliability (consume SLOs).

### Cross-cutting enabler — the Self-Reinforcing RAG Knowledge Base
*Goal: turn the platform's own usage into what makes its agents smarter.*

**Why we have the data.** The platform is the **central data hub** — *every*
query, and increasingly every **intent** (§3), lands and is processed here — so we
sit on a uniquely rich corpus:

- **What we capture:** what was asked, by whom, over which datasets, how it was
  planned and routed, what it cost, and whether it met its SLA.
- **How agents use it:** we keep that corpus as an **in-house knowledge base** and
  let agents **retrieve over it (RAG)** to ground decisions in what has actually
  worked here before — not generic heuristics.

**What it feeds** — the AI-driven parts already in this plan:

- **Query optimization** — retrieve similar past queries and their winning plans
  to guide execution.
- **Query / data routing** — the ILM promotion decision (Pillar 1) and the
  bi-modal routing (§5) consult prior outcomes to pick Doris / Trino / cache /
  native copy.
- **Autonomous operations** — past incidents, tuning actions and their results
  become retrievable playbooks for the ops / steward agents (Pillar 4).
- **Infra scaling** — historical load and spike patterns inform capacity and
  pre-scaling decisions.
- **Engine-choice guidance** — the very judgment the professional-services team
  makes today (§2) becomes a retrievable, self-serve recommendation.

**Why it compounds.** The loop is self-reinforcing: more usage → a richer
knowledge base → better agent decisions → a better platform → more usage. It is
the platform applying **"data as the differentiator" (Pillar 2) to itself** — our
own operational telemetry is what makes our agents better than any off-the-shelf
model could be alone.

---

## 5. Architectural Blueprint — The 6-Layer Stack

This is the **concrete realization** of the four pillars — the physical layer
stack. It builds on prior reference architectures but **converges** what they
keep separate:

- **Adapts** a16z's 2020 *Unified Architecture* and *AI/ML blueprint* (ref [5]) —
  but **collapses** the heavy decoupled ingestion pipelines and isolated ML
  model-serving clusters into a single platform where **LLMs actively operate the
  infrastructure layer** (Meta-scale internal-platform reference, ref [6]).
- **Draws on** the AWS *Open Data Foundation for Agentic AI* whitepapers
  (Iceberg-native, zero-ETL) and the Databricks *Data Intelligence Platform*
  reference architecture (convergence under an AI-driven catalog).

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 1 · DATA SOURCES                                                   │
│     Legacy Hadoop Suite  ·  Oracle RAC (pending)  ·  Real-time event logs│
└──────────────────────────────────────────────────────────────────────────┘
          │   Zero-ETL / Federated / Spark ingestion
          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 2 · AGENTIC CONTROL PLANE & SEMANTIC LAYER                         │
│     replaces standalone catalogs + manual pipeline orchestration         │
│     • LLM auto-routing        • Privacy / compliance guardrails          │
│     • Universal semantic layer — one metric definition for BI = ML       │
│     • AI-driven catalog (Databricks-style convergence)                   │
└──────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 3 · UNIFIED STORAGE — THE ZERO-LOCK-IN LAKEHOUSE                   │
│     replaces proprietary warehouses + fragmented data lakes              │
│     • Open table format: Apache Iceberg     • Object storage: MinIO      │
│     • AI-driven ILM: lifecycle + cache, one Iceberg copy (100+ PB)       │
└──────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 4 · UNIFIED PROCESSING & AGENTIC AI ENGINE                         │
│     merges a16z processing layer with the AI/ML training blueprint       │
│     • Analytics compute: Apache Spark (5000+ cores)                      │
│     • AI orchestration: LLMs operating Hyperopt for auto-tuning          │
│     • Model training: Spark MLlib & XGBoost (distributed heavy lifting)  │
│     • Infra ops: Celery-managed k6 workers for native load testing       │
└──────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 5 · SERVING & QUERY ENGINE   (bi-modal, control-plane routed)      │
│     replaces monolithic one-size query engines                           │
│     • Real-time OLAP: Apache Doris (system metrics, fast-refresh BI)     │
│     • Ad-hoc massive scale: Trino (cross-cluster, multi-table federated) │
│     → one Iceberg source of truth; native copy only for real-time BI     │
└──────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ LAYER 6 · OUTPUT & APPLICATIONS                                          │
│     evolves the a16z output layer beyond static dashboards               │
│     • Accessible analytics (real-time charts, semantic reports)          │
│     • Autonomous AI data scientists (LLMs querying data directly)        │
│     • Agentic AI apps (RAG-native workflows over Iceberg)                │
└──────────────────────────────────────────────────────────────────────────┘
```

### Layer → pillar → persona mapping

| Blueprint layer | Maps to pillar(s) | Lead persona |
|---|---|---|
| 1 · Data Sources | inputs to P1 | 📦 Data Owners onboard |
| 2 · Agentic Control Plane & Semantic Layer | **P2 + P4** (fused) | 🛠️ Platform builds · 📦 Owners define · agents enforce |
| 3 · Unified Storage (zero-lock-in lakehouse) | **P1** | 🛠️ Platform · 📦 Owners onboard |
| 4 · Unified Processing & Agentic AI Engine | **P3** (+ P4 ops) | 🛠️ Platform · 🔍 Users |
| 5 · Serving & Query Engine (bi-modal) | **P1** serving | 🛠️ Platform (agents route) |
| 6 · Output & Applications | Consumers | 🔍 Data Users |

### What we retire vs. what replaces it

The blueprint is defined as much by what it *removes* as what it adds — heavy,
decoupled 2020-era components are truncated and folded into the converged stack.

| Layer | Retired / truncated (2020-era) | Replaced by |
|---|---|---|
| **2** | Standalone data catalogs, manual Airflow/ETL orchestration, isolated metrics layers | Active **LLM-driven control plane** enforcing one semantic layer across Doris, Trino & ML — e.g. a single definition of "monthly active users" everywhere |
| **3** | Proprietary warehouse storage; the hard Lake ↔ Warehouse divide | **Apache Iceberg on MinIO** as the single source of truth; **AI-driven ILM** keeps one Iceberg copy and tiers / caches by access pattern + latency SLA (object lifecycle · SSD cache · bounded native copy) across **100+ PB** |
| **4** | Independent MLOps training stack (Kubeflow, standalone MLflow); isolated heavy-transform tools (dbt) | **Spark does both** transform & training (MLlib/XGBoost); **LLMs operate Hyperopt** to run experiments, solve cold-start & auto-train; **Celery-managed k6** load testing baked into the **5000+ core** tier |
| **5** | Generalized one-size serving engines (e.g. Presto as both BI + batch) | **Bi-modal, control-plane-routed:** Doris for real-time / fast-refresh BI, Trino for massive ad-hoc history — **both query the same Iceberg tables (one source of truth)**; only the few real-time / high-QPS dashboards get a thin, auto-fed Doris-native copy |
| **6** | Static BI dashboards | Accessible analytics, **autonomous AI data scientists**, and **RAG-native agentic apps** over Iceberg |

> **Note on topology:** This blueprint draws the **Agentic Control Plane fused
> with the Semantic Layer as Layer 2** (a horizontal interception layer above
> storage), whereas the pillar view (§4) treats the control plane as
> **cross-cutting (Pillar 4)** and the semantic layer as **Pillar 2**. Same
> capabilities, two lenses — the blueprint stresses that *every* request passes
> through agent governance, and makes the **bi-modal serving split (Layer 5)**
> explicit.

### Query routing & dialect handling (how bi-modal serving actually works)

"Bi-modal, control-plane-routed" only works because Doris and Trino are **not
SQL-compatible** (Doris is MySQL-protocol / MySQL-dialect — backtick identifiers,
`DATE_FORMAT`, `APPROX_COUNT_DISTINCT`; Trino is ANSI / Presto — double-quote
identifiers, `format_datetime`, `approx_distinct`, lambdas). So the control plane
**never routes a raw dialect-specific SQL string between engines.** It routes
*logical intent* — which is exactly why Layer 2 fuses the control plane **with**
the semantic layer. Three mechanisms, layered:

1. **Semantic-layer compilation — primary.** Users / agents / BI tools query the
   semantic layer, not a raw engine. The logical model is defined **once** and a
   **per-engine compiler emits native Doris SQL *or* native Trino SQL**. The
   router picks the engine; the semantic layer produces the dialect.
   *Compatibility is solved by generation, not translation* (cf. dbt MetricFlow,
   Cube, Malloy, ref [16]).
2. **Canonical dialect + transpiler — secondary (raw SQL).** Power users / legacy
   tools standardize on **one canonical dialect (Trino ANSI)**. Routed to Trino →
   pass through; routed to Doris → **transpile** (SQLGlot, ref [14]; or Apache
   Calcite, ref [15]) — best-effort, with the fallback below on any unsupported
   construct.
3. **Route by data product — the pragmatic shrink.** The Doris-native path serves
   **curated dashboards / metrics with generated, known SQL**; arbitrary human
   ad-hoc defaults to Trino. Very few queries ever need cross-engine portability.

**Two guardrails:**
- **Trino-on-Iceberg is the always-correct fallback.** If the router can't safely
  compile / transpile to the fast engine (or the dataset has no Doris serving
  copy), it defaults to Trino reading Iceberg. *Correctness beats latency — never
  silently return a wrong result.*
- **Consistency comes from the semantic layer, not identical SQL.** Both engines
  derive from the same Iceberg source of truth and the same metric definitions,
  so "monthly active users" returns the same number whether compiled to Doris or
  Trino — even though the SQL text differs.

> **Net:** transparent cross-engine routing of *raw* SQL is not realistically
> achievable — bi-modal serving is viable only because the **semantic layer sits
> in front of both engines** as the entry point.

---

## 6. Online / Product-Facing Serving (the missing online path)

**The gap.** Scenarios 1–3 are all *offline*; none serves a live product **in the
request path** — this scenario (4) adds exactly that:

- **Offline today (scenarios 1–3):** humans reading dashboards, humans running
  ad-hoc queries, and ML **training**.
- **Online, new (scenario 4):** the platform serves application traffic directly —
  à la Uber's apps reading fresh platform data live, plus real-time ML
  **inference** (the inference half that training alone doesn't cover).

> **Relation to the blueprint (§5):** this online path *extends* Layer 5 (serving)
> and Layer 6 (output) with a **streaming backbone** (Kafka → Flink) and an
> **online / feature store**, and adds **product / application** as a new consumer.

### Why it's a fundamentally different problem

| Dimension | Offline (scenarios 1–3) | Online / product-facing (scenario 4) |
|---|---|---|
| Latency | seconds → minutes | **p99 ≈ 10–100 ms** |
| Concurrency | tens of human users | **thousands → millions QPS** |
| Availability | best-effort | **99.9 %+ — it's in the revenue / request path** |
| Consumer | humans, batch trainers | **applications, end users, AI agents** |
| Freshness | hourly / daily | **seconds (streaming)** |
| Isolation | shared capacity is fine | **must be isolated from ad-hoc load** |

You cannot meet the right-hand column by pointing the product at Trino-on-Iceberg
or a human BI engine — it needs **purpose-built serving tiers**.

### Two serving sub-patterns

1. **Online *analytical* serving** — aggregations / metrics rendered inside the
   product (embedded dashboards, live counters, "who-viewed-your-X"). Served by a
   **real-time OLAP** engine: **Doris** / Apache Pinot / StarRocks — purpose-built
   for user-facing, high-concurrency analytics (Pinot was built at LinkedIn for
   exactly this).
2. **Online *point / feature* serving** — millisecond key lookups: ML **feature
   serving for inference**, personalization, risk scoring. Served by a low-latency
   **online / feature store** on a KV backend: **Cassandra** / Redis / Databricks
   Lakebase. Its job is to keep online features identical to the offline ones used
   in training — **no train/serve skew**.

### Reference architecture (streaming-fed, Iceberg-reconciled)

```text
  app / CDC events
        │
        ▼
  Apache Kafka            ·  durable event log
        │
        ▼
  Flink / stream proc.    ·  enrich, aggregate, compute features
        │
        ├──────────────▶  Real-time OLAP: Doris / Pinot / StarRocks
        │                    → in-product analytics, live metrics, embedded dashboards
        │
        └──────────────▶  Online store: Cassandra / Redis / Lakebase feature store
                             → ML inference, personalization, point lookups
                                       │
                                       ▼
                             PRODUCT / APP   (ms request path, high QPS)

  Iceberg on MinIO  =  offline source of truth  (training · ad-hoc · BI)
        ▲
        └─ same events also land here; online tiers are materialized / reconciled
           from Iceberg   ⇒   train/serve & batch/online consistency
```

### How it maps onto *our* stack (largely a reframing, not a rebuild)

- **Cassandra → promoted from "operational, not for analysis" to the online
  serving / feature-store tier.** It is already a canonical low-latency KV backend
  for exactly this; it becomes a first-class part of the platform.
- **Doris → extends from internal BI to external, product-facing real-time
  analytics.** It is in the same MPP real-time-OLAP family as StarRocks (which
  began as a Doris fork) — the engine class used for user-facing analytics.
- **Iceberg on MinIO stays the offline source of truth;** online tiers are
  **materialized / reconciled from it** — the same *"single source of truth ≠
  single physical copy"* principle as the ILM model (§4, Pillar 1), here applied
  as **train/serve & batch/online consistency**.
- **Streaming backbone (Kafka → Flink)** feeds the online tiers *and* lands in
  Iceberg (Kappa-style), so online and offline see one event stream.
- **The same semantic layer governs both** — a metric or feature has one
  definition whether served to a dashboard, a model, or the product (§5 routing /
  consistency principle).

### New consumer

This adds **product / application engineering teams** as a consumer — distinct
from the human *Data Users* (analysts / scientists). They integrate **serving
endpoints / SDKs** into the product, not SQL notebooks.

### Architectural guardrails

- **Hard workload isolation.** Product serving must not share capacity with ad-hoc
  analyst queries — separate clusters / tenants, dedicated serving replicas. (Uber
  even exposes a *direct-connection mode* that bypasses its query proxy for
  isolation.)
- **Provision for peak + 99.9 %+ SLA** — it's revenue-impacting; capacity and
  on-call posture differ from the offline world.
- **Freshness contract** — define per data product how stale online data may be
  (seconds vs. minutes) and back it with the streaming pipeline.

### Real-world cases (researched)

- **Uber — Apache Pinot:** powers hundreds of user-facing analytics use cases;
  >90 % expect **sub-second** latency, QPS from single digits to **many
  thousands**, tables with **hundreds of billions** of records [17].
- **LinkedIn — Apache Pinot:** built at LinkedIn to serve **"Who viewed your
  profile"** and similar features to millions of external users [18].
- **Online feature stores (Feast / Tecton):** offline store (lake / warehouse) +
  online KV (Redis / DynamoDB / **Cassandra**) with shared definitions for **<10
  ms** serving and no train/serve skew [19].
- **Databricks Lakebase (GA 2025):** managed Postgres "operational database for AI
  apps and agents," powering low-latency online feature serving in the lakehouse
  [20].
- **Real-time OLAP for customer-facing analytics** (Pinot / Druid / ClickHouse /
  StarRocks): Pinterest's migration Druid → StarRocks reportedly cut infra cost
  ~3× and p90 latency ~50 % [21]; Lambda / Kappa streaming underpins all of these
  [22].

---

## 7. Summary Table

| Pillar / Capability | From (today) | To (vision) | Key tech | Persona focus |
|--------|--------------|-------------|----------|---------------|
| 1. Foundation | Separate lake + warehouse, manual ETL from Oracle | Converged Zero-ETL lakehouse with AI-driven ILM | Iceberg, Trino federation, Doris/ClickHouse, MinIO | 🛠️ Platform builds · 📦 Owners onboard |
| 2. Semantic | Physical tables | Global semantic layer / accessible analytics | Central semantic layer, lineage, embedded privacy | 🛠️ Platform builds · 📦 Owners define · 🔍 Users consume |
| 3. Data Science | Separate ML platform, legacy training data | Unified agentic Data + AI hub | LLM agents + XGBoost/Spark MLlib + Hyperopt | 🛠️ Platform builds · 🔍 Users build models |
| 4. Operations | Manual tuning & monitoring | Autonomous ops & stewardship | Agent auto-tuning, Celery load testing, steward agents | 🛠️ Platform builds & operates · 📦 Owners remediate |
| 5. Online serving *(new)* | Offline only (human / batch) | Product-facing real-time serving + ML inference | Kafka / Flink, Doris / Pinot, Cassandra / Redis / Lakebase | 🛠️ Platform builds · 📱 Product teams consume |

---

## 8. Guiding Principles

1. **Unify, don't bolt on** — one platform for Data *and* AI (org implication:
   eventual convergence with the separate ML Platform team).
2. **All-build on open source** — Iceberg / open tables and OSS engines keep the
   stack swappable and avoid **vendor lock-in** (company policy: build, don't buy).
3. **Meaning over movement** — invest in the semantic layer; it's the foundation
   for both BI trust and agent correctness.
4. **Agents orchestrate, proven engines compute** — LLMs reason and coordinate;
   distributed ML/SQL engines do the heavy math.
5. **Autonomy with guardrails** — agents optimize and steward, but within
   encoded privacy/compliance/quality constraints.
6. **The platform's telemetry is the agents' fuel** — every query and intent
   feeds an in-house **RAG knowledge base**, so routing, optimization, ops and
   scaling decisions **compound with usage** rather than relying on generic
   heuristics (the self-reinforcing RAG Knowledge Base).

---

## 9. Open Questions & Decisions

### Decided
- **Build vs. buy → ALL BUILD.** Company policy avoids **vendor lock-in**, so the
  platform is assembled entirely from **open source** on our existing
  Iceberg / Trino / Doris / Spark / Kafka base. Databricks and other vendors
  inform the *concepts and reference architectures only* — we do not buy the
  products.
- **Ownership → US.** The **Platform Team (our Big-Data team) owns the unified
  Data & AI platform**; the ML Platform's capabilities fold into it under our
  ownership (convergence timing is in the roadmap).

### Still open
- **Semantic layer choice (OSS):** Which technology powers the global semantic layer,
  and how does it integrate with Trino + Doris?
- **Zero-ETL scope:** Which legacy sources (Oracle first?) get Zero-ETL /
  federation, and in what order?
- **Agent trust & safety:** What is the approval/guardrail model for autonomous
  agents touching production data and infra?
- **Real-time path:** Doris vs. ClickHouse (or both) for the hot OLAP tier
  serving real-time BI.
- **Data Owner enablement:** what self-service onboarding and semantic-authoring
  tooling do Data Owners need to register, document, and certify datasets
  without depending on the Platform Team for every change?
- **Online serving — engine choice (OSS):** for product-facing real-time
  analytics, extend **Doris** or adopt Pinot / StarRocks? For the online
  feature-store backend, **Cassandra** vs. Redis vs. self-managed Postgres
  (managed options like Lakebase are excluded by the all-build policy)?
- **Serving isolation & SLA:** how do we isolate revenue-path serving from ad-hoc
  analytics (separate clusters / tenants), and what latency / availability SLAs do
  product teams get?
- **Professional-services → automation transition:** how do we encode the services
  team's engine-selection and prototyping judgment into the semantic layer +
  control plane, and what becomes of the team once auto-routing exists (shift from
  per-request help to platform / agent curation)?

---

## 10. Reference Sources

### 10.1 Extended Materials — AWS Agentic AI & Data Foundation (2025–2026)
These shaped the **"Unified Storage"** (Pillar 1) and **"Agentic Control Plane"**
(Pillar 4) layers and reflect the latest agent-operated-systems thinking.

1. **Data Foundation for Agentic AI — Open Architecture (AWS).**
   An open data foundation lets AI agents *find, access, trust, and act on* data
   across systems; avoid lock-in via open-source tech, open table formats, and
   managed services; **Iceberg** provides the cross-engine open table format.
   → `https://aws.amazon.com/data/`
2. **Architecting for Agentic AI Development on AWS (AWS Architecture Blog).**
   Systems must be redesigned for agents; **lightweight cloud testing + preview
   environments + domain-driven structure** create an environment where agents
   operate effectively and safely.
   → `https://aws.amazon.com/blogs/architecture/architecting-for-agentic-ai-development-on-aws/`
3. **Rethinking SaaS in the Agentic Era (AWS Whitepaper).**
   Business/technical/operational implications of agent-enabled systems; the
   evolution from **reactive assistants → proactive, autonomous systems** that
   understand, decide, and act with minimal oversight.
   → `https://d1.awsstatic.com/onedam/marketing-channels/website/aws/en_US/isv/approved/images/en_us__aws_isv_rethinking-saas-in-the-agentic-era_whitepaper.pdf`
4. **AWS — The Generative AI White Paper.**
   **Data as the differentiator** for gen-AI applications; a strong data
   foundation and strategic data management are crucial to unlocking gen-AI value.
   → `https://www.scribd.com/document/932717584/AWS-The-generative-AI-White-Paper`

### 10.2 Foundational Materials (original brief)
5. **Emerging Architectures for Modern Data Infrastructure (a16z, 2020).**
   Structural baseline for moving from fragmented data lakes to unified
   architectures; Unified Architecture diagram + Modern BI blueprint.
   → `https://a16z.com/emerging-architectures-for-modern-data-infrastructure-2020/`
6. **Data Engineering at Meta — High-Level Overview of the Internal Tech Stack.**
   Reference model for scaling internal platforms.
   → `https://medium.com/@AnalyticsAtMeta/data-engineering-at-meta-high-level-overview-of-the-internal-tech-stack-a200460a44fe`
7. **The Future of the Data Engineer (Meta).**
   Moving away from manual ETL toward **accessible analytics** and logical data
   management.
   → `https://medium.com/@AnalyticsAtMeta/the-future-of-the-data-engineer-part-i-32bd125465be`
8. **Data Lakes and Analytics on AWS.**
   AWS's high-level vision for merging data lakes with BI and ML tools.
   → `https://aws.amazon.com/big-data/datalakes-and-analytics/`
9. **Databricks Data Intelligence Platform — reference architecture (concept).**
   Core inspiration for merging the Data and ML/AI platforms into one
   workspace; its **AI-driven catalog** drives complete convergence — one
   governed catalog spanning BI and ML.
10. **Gemini chat history (prior brainstorm).**
    Conceptual basis for using LLMs as autonomous agents to handle model training
    and optimization workflows natively.

### 10.3 ILM, tiering & caching primitives (grounding for the corrected ILM model)
11. **AWS S3 Intelligent-Tiering / S3 Lifecycle.**
    Automated object tiering by access pattern; **format-agnostic** — the
    object's storage class changes, not its table format. Grounds ILM **Level A**.
    → `https://aws.amazon.com/s3/storage-classes/intelligent-tiering/`
12. **Databricks — Predictive Optimization & disk cache.**
    AI-driven table optimization (auto compaction / clustering) plus local-NVMe
    caching of remote data — speed without a second format. Grounds ILM **A & B**.
    → `https://docs.databricks.com` (features: *Predictive Optimization*, *disk cache*)
13. **Apache Doris / StarRocks — external-catalog data cache.**
    Local-disk cache for Iceberg / Hive external tables that approaches native
    latency while querying a single Iceberg copy. Grounds ILM **Level B**.
    → `https://doris.apache.org/docs/lakehouse/` · `https://docs.starrocks.io`

### 10.4 Query routing & dialect compilation (grounding for bi-modal serving)
14. **SQLGlot — multi-dialect SQL parser & transpiler.**
    Parses Trino / Presto and emits MySQL / StarRocks-family dialects; the
    best-effort transpiler for the raw-SQL path. Grounds routing mechanism 2.
    → `https://github.com/tobymao/sqlglot`
15. **Apache Calcite — query planning & multi-dialect SQL generation.**
    Framework for parsing, optimizing, and generating SQL across dialects; an
    alternative engine for dialect compilation / federation.
    → `https://calcite.apache.org/`
16. **Semantic-layer compilers — dbt MetricFlow · Cube · Malloy.**
    Define metrics once; generate native per-engine SQL. Grounds routing
    mechanism 1 (compilation, not translation).
    → `https://docs.getdbt.com/docs/build/about-metricflow` · `https://cube.dev` · `https://www.malloydata.dev`

### 10.5 Online / product-facing serving (real-world cases)
17. **Uber — Rebuilding Uber's Apache Pinot Query Architecture.** Real-time,
    user-facing analytics at scale: sub-second latency, thousands of QPS,
    hundreds of billions of records.
    → `https://www.uber.com/us/en/blog/rebuilding-ubers-apache-pinot-query-architecture/`
18. **Apache Pinot (built at LinkedIn).** Purpose-built for ultra-low-latency,
    high-concurrency user-facing analytics (origin: "Who viewed your profile").
    → `https://pinot.apache.org/`
19. **Feature stores — Feast / Tecton.** Offline + online architecture; online KV
    (Redis / DynamoDB / Cassandra); shared definitions prevent train/serve skew.
    → `https://feast.dev` · `https://www.tecton.ai`
20. **Databricks Lakebase + Online Feature Stores (GA 2025).** Managed Postgres
    operational DB for AI apps / agents; low-latency online feature serving.
    → `https://www.databricks.com/blog/databricks-lakebase-generally-available`
21. **Real-time OLAP for customer-facing analytics (Pinot · Druid · ClickHouse ·
    StarRocks).** Engine trade-offs for high-QPS, low-latency serving; Pinterest
    Druid → StarRocks reported ~3× cost and ~50 % p90 latency improvement.
    → `https://startree.ai/resources/a-tale-of-three-real-time-olap-databases/`
22. **Lambda / Kappa streaming architecture.** Kafka log + Flink stream processing
    feeding low-latency serving stores (Cassandra / Redis) for product features.
    → `https://www.kai-waehner.de/blog/2025/07/08/the-rise-of-kappa-architecture-in-the-era-of-agentic-ai-and-data-streaming/`

### 10.6 RAG & learned optimization (grounding for the Knowledge Base)
23. **Retrieval-Augmented Generation (Lewis et al., 2020).** Grounding LLM
    outputs in a retrieved knowledge corpus — the basis for the in-house
    knowledge base built from query + intent telemetry.
    → `https://arxiv.org/abs/2005.11401`
24. **Learned query optimization (e.g. MIT Bao / Neo).** Using past execution
    feedback to improve query planning & routing — grounds the optimization /
    routing use cases of the knowledge base.
    → `https://arxiv.org/abs/2004.03814` (Bao)
