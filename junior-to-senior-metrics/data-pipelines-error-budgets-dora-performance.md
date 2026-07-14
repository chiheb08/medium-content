# The Pipeline Promised Fresh Data by 8 a.m. — Error Budgets, DORA, and Reviews for Data Teams

**Part 2 of:** *Stop Optimizing Your Ticket Count: The Metrics That Change When You Go From Junior to Senior*

**Alternative titles for Medium:**
- *Error Budgets for Data Pipelines — A Practical Follow-Up for Data Engineers*
- *DORA Metrics When Your “Deploy” Is a DAG Run*
- *How to Talk About Pipeline Reliability in Your Performance Review*

**Subtitle:** Freshness SLOs, job failure rates, recovery time, and the language that makes data engineering look senior — not just busy.

> **Medium tip:** Paste section headings as **H2** blocks. Use the **“At a glance”** list as your scroll map. For tables, screenshot from the full `.md` file or use the `*-medium-no-tables.md` companion.

---

## At a glance — what this follow-up gives you

- **Error budgets in practice** for data pipelines — freshness, completeness, and when to ship risky changes
- **DORA metrics translated** for data teams (deployment frequency = pipeline releases; CFR = failed runs / bad publishes)
- **Pipeline-native reliability:** MTTR for backfills, MTBF for chronic flakiness, late data, schema drift
- **How to present impact** in performance reviews without listing every ticket
- **Junior vs senior** habits specific to batch, streaming, and hybrid stacks

**One line to remember:** A green Airflow task is not a promise — an **SLO with an error budget** is.

**Who this is for:** data engineers, analytics engineers, and platform folks who own **pipelines, warehouses, and SLAs to downstream teams** — not generic product on-call (though the ideas rhyme).

> **New to the jargon?** Read **[Terms defined — data pipeline dictionary](#terms-defined--data-pipeline-dictionary)** first. Shared metrics (SLO, MTTR, DORA, etc.) are defined in [Part 1’s dictionary](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md#terms-defined--the-dictionary-read-this-first).

---

## Terms defined — data pipeline dictionary

Terms that confuse **non-data** folks — or juniors hearing them for the first time.

### Core ideas

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Data pipeline** | Automated path that **moves and transforms** data from sources (apps, databases) to destinations (warehouse, dashboards) | Your job is not “run a script” — it is **keep a promise** to downstream users. |
| **ETL / ELT** | **Extract** data → **Transform** → **Load** (or Load then Transform in the warehouse) | Naming varies; consumers care about **output**, not acronym order. |
| **Data warehouse** | Central store for **analytics** (BigQuery, Snowflake, Redshift, etc.) | Where “the number” on the dashboard usually lives. |
| **Mart / data mart** | A **table or dataset** built for a team or use case (e.g. “daily revenue mart”) | Often what **SLAs** attach to. |
| **Consumer / downstream** | Team or system that **reads** your output | They do not see your green task — they see **wrong charts**. |
| **Upstream** | System that **produces** data you depend on | Late upstream = late you — negotiate **their** SLA too. |

### Tools you will hear (not required to master every one)

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Airflow** | Scheduler that runs workflows on a **calendar** or trigger | Green square = task ran — **not** always “data is correct.” |
| **DAG** | **Directed Acyclic Graph** — a workflow diagram of tasks with dependencies (A before B) | “The DAG failed” = some step in the chain broke. |
| **dbt** | Tool to **transform** data in the warehouse using SQL + tests | “Deploy” often means **merge models** to production target. |
| **Spark** | Engine for **large-scale** batch (and sometimes stream) processing | Cluster can be “busy” while **freshness** still misses. |
| **Partition** | Splitting a table by **date or key** (e.g. `dt=2026-07-14`) | Enables **rerun one day** without rebuilding everything. |
| **Backfill** | **Reprocessing** past dates after a bug or schema fix | Recovery move — measure **MTTR** including backfill time. |

### Data quality and shape

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Freshness** | How **up to date** the data is | “Revenue as of when?” — the #1 pipeline question. |
| **Completeness** | Whether **all expected rows** arrived | Silent row loss is worse than a loud failure. |
| **Correctness** | Whether values follow **business rules** (no dup keys, valid ranges) | Tests exist to catch this **before** publish. |
| **Schema** | **Column names and types** of a dataset | Drift = upstream changed without telling you. |
| **Schema drift** | Schema changed **unexpectedly** (column renamed, type changed) | Classic cause of **silent wrong data**. |
| **dbt test / data quality check** | Automated rule: “this column unique,” “no nulls,” etc. | Failing test should **block** bad publishes. |
| **Lineage** | **Where data came from** — upstream tables and jobs | Explains “why is this number wrong?” faster. |

### Metrics from Part 1 (quick reminder)

| Term | One-line reminder |
|------|-------------------|
| **SLI** | What you **measure** (e.g. “loaded before 8 a.m.”) |
| **SLO** | Target you **aim for** (e.g. “99% of days on time”) |
| **SLA** | **Contract** with someone outside the team |
| **Error budget** | How much lateness/failure you can **afford** per month |
| **MTTR** | How fast you **fix** a bad or late dataset |
| **MTBF** | How often the **same** pipeline breaks again |
| **DORA / CFR** | How often **changes** to pipelines cause harm |
| **ROI** | Whether the pipeline **earned** its engineering cost |

---

## Table of contents

1. [Terms defined — data pipeline dictionary](#terms-defined--data-pipeline-dictionary)
2. [Why data pipelines need their own sequel](#why-data-pipelines-need-their-own-sequel)
2. [Why data pipelines need their own sequel](#why-data-pipelines-need-their-own-sequel)
3. [The data pipeline promise — what you are actually selling](#the-data-pipeline-promise--what-you-are-actually-selling)
4. [SLIs and SLOs for pipelines — pick the right promise](#slis-and-slos-for-pipelines--pick-the-right-promise)
5. [Error budgets in practice — spending trust on purpose](#error-budgets-in-practice--spending-trust-on-purpose)
6. [When to burn budget vs when to freeze changes](#when-to-burn-budget-vs-when-to-freeze-changes)
7. [DORA for data teams — same ideas, different nouns](#dora-for-data-teams--same-ideas-different-nouns)
8. [Deployment frequency — shipping pipeline change safely](#deployment-frequency--shipping-pipeline-change-safely)
9. [Change failure rate — when a merge hurts consumers](#change-failure-rate--when-a-merge-hurts-consumers)
10. [Lead time and MTTR — recovery in data land](#lead-time-and-mttr--recovery-in-data-land)
11. [MTBF — pipelines that break every Tuesday](#mtbf--pipelines-that-break-every-tuesday)
12. [Metrics dashboards data seniors actually watch](#metrics-dashboards-data-seniors-actually-watch)
13. [Performance reviews — prove impact without ticket spam](#performance-reviews--prove-impact-without-ticket-spam)
14. [A quarter plan template for pipeline owners](#a-quarter-plan-template-for-pipeline-owners)
15. [Conclusion — the senior data engineer sentence](#conclusion--the-senior-data-engineer-sentence)

---

## Why data pipelines need their own sequel

In [Part 1](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md), we mapped how thinking shifts from **“my task is done”** to **“the system stays trustworthy.”** For data teams, that shift has a twist:

> Your users often never open your repo. They open a **dashboard**, a **ML feature store**, or a **finance spreadsheet** — and assume the number is true.

**Junior pipeline instinct:** “The job succeeded.”  
**Senior pipeline instinct:** “Did the **right data** arrive **on time**, **complete**, and **understood** by consumers?”

**Real-life analogy:** A bakery’s oven light says **“done”** (task success). Customers care if the **bread arrived at the café by 7 a.m.**, **all loaves**, **not yesterday’s batch** (SLO).

This article is the **data-pipelines** follow-up: error budgets, DORA, and performance reviews — with examples you can steal for your next 1:1.

---

## The data pipeline promise — what you are actually selling

Every pipeline makes an implicit promise. Seniors make it **explicit**.

| Promise type | What the consumer hears | Example SLI |
|--------------|-------------------------|-------------|
| **Freshness** | “Data is current enough for decisions.” | `max(event_time)` landed before 08:00 UTC |
| **Completeness** | “We didn’t silently lose rows.” | Row count within 2% of source |
| **Correctness** | “Keys and business rules hold.” | 0% duplicate `order_id`; refunds ≤ sales |
| **Availability** | “The table exists and is queryable.” | Table readable 99.9% of the month |
| **Lineage clarity** | “I know where this number came from.” | Documented owner + upstream deps |

**Pitfall:** Teams optimize **job success rate** while **freshness SLO** is on fire — like celebrating an on-time flight that landed in the wrong city.

---

## SLIs and SLOs for pipelines — pick the right promise

### SLI — Service Level Indicator

**Defined:** A **measurable signal** of pipeline health — the specific number you track (e.g. “minutes late,” “% jobs failed”).

Examples:

- **Freshness lag:** `now() - max(partition_date)` for a daily mart
- **Job success:** % of scheduled runs succeeded without retry
- **Data quality:** % of checks passed (Great Expectations, dbt tests, Deequ)
- **End-to-end latency:** event time → warehouse availability (critical for streaming)

### SLO — Service Level Objective

**Defined:** The **target** you commit to hit for that SLI over time (e.g. “on time 99% of days per month”).

**Example (daily revenue mart):**

- **SLI:** data for day *D* available by **06:00 UTC** on day *D+1*
- **SLO:** meet that deadline on **99%** of days per rolling month

### SLA — when finance or product is in the contract

Maybe: “Executive dashboard refreshed by 8 a.m. local **or** incident declared.”

**Real-life analogy:** **Newspaper deadline** — the printing press working (job green) is not the same as **papers on doorsteps by 6 a.m.**

### Junior → senior shift (pipeline SLOs)

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “All tasks green in Airflow.” | “Do **consumers** get trustworthy data on time?” |
| “One SLO for everything.” | **Different SLOs** per mart — revenue vs marketing vs ML features |
| “We’ll add monitoring later.” | SLO defined **before** the dashboard goes viral |

---

## Error budgets in practice — spending trust on purpose

An **error budget** is how much unreliability your SLO allows before you are “out of policy.”

### Simple freshness example

**SLO:** 99% on-time daily loads in a 30-day month → you can miss the deadline on **~0.3 days** (~7 hours total slack, depending how you measure) — or roughly **1 day** if you round to whole days in policy.

That slack is not “free vacation.” It is **currency** for:

- planned risky migrations,
- vendor outages,
- holiday traffic spikes,
- intentional late loads when **upstream** was wrong and you refused to publish garbage.

**Real-life analogy:** A monthly phone data plan with **1% roaming buffer** — you can use it for a real trip, not to stream movies daily and act shocked at the bill.

### How to compute budget burn (conceptual)

For a **binary** daily SLO (on-time vs late):

$$
\text{budget remaining} \approx 1 - \frac{\text{missed days}}{\text{allowed missed days}}
$$

For **time-based** SLO (minutes late):

$$
\text{burn} = \frac{\text{total lateness minutes this month}}{\text{monthly lateness allowance}}
$$

*In plain English:* are we **ahead** or **behind** on our monthly “oops allowance”?

### Pipeline-specific budget spenders

| Spend type | Deliberate? | Example |
|------------|-------------|---------|
| **Schema migration** | Often yes | Breaking change with comms + backfill window |
| **Upstream delay** | Sometimes | Source API late — you hold publish to protect correctness |
| **Bad deploy** | No | CFR event — burns budget + trust |
| **Retry storm** | No | Masked flakiness — fix MTBF, not just re-run |
| **Manual override** | Gray | “Ship partial data” — document and measure |

**Senior move:** Log **why** budget burned in the incident channel — “late because Stripe export slipped” vs “late because we had no alert.”

---

## When to burn budget vs when to freeze changes

Error budgets are useless without **policy**. A practical playbook:

### Green zone — budget healthy

- Ship refactors, partition changes, cost optimizations
- Experiment with new orchestration features
- Accept small known risks with rollback plans

### Yellow zone — budget thinning

- Freeze **non-essential** schema changes
- Increase monitoring granularity (partition-level freshness)
- Pre-mortem on upcoming heavy days (Black Friday, month-end close)

### Red zone — budget exhausted or SLA at risk

- **Change freeze** on critical paths
- Daily standup on recovery only
- Escalate upstream SLAs with evidence (“we missed 4/4 days because source landed at 11:00”)

| Zone | Budget left (illustrative) | Pipeline team behavior |
|------|----------------------------|-------------------------|
| Green | > 50% | Normal velocity |
| Yellow | 20–50% | Caution — smaller batches |
| Red | < 20% or SLA threatened | Stabilize — MTTR focus |

**Real-life analogy:** Hospital ER — when beds are full, **elective surgeries pause**. Not because surgery is bad, because **trust and capacity** are finite.

### The conversation that sounds senior in a meeting

> “We have **18% error budget left** on revenue mart freshness. I recommend we **defer** the dimension table refactor until next sprint and ship only the **null-rate fix** that stops bad joins.”

That is not bureaucracy. That is **risk management in plain language**.

---

## DORA for data teams — same ideas, different nouns

[DORA metrics](https://dora.dev/) were framed for **application delivery**. Data platforms map cleanly if you translate nouns:

| DORA metric | Classic software | Data pipeline translation |
|-------------|------------------|---------------------------|
| **Deployment frequency** | Prod releases | **Pipeline releases** — dbt merges, DAG deploys, Flink job updates |
| **Lead time for changes** | Commit → prod | PR merged → **production dataset** reflects change |
| **Change failure rate (CFR)** | Bad deploys | Runs that **fail** or publish **bad data** requiring rollback/backfill |
| **MTTR** | Restore service | Time to **correct data** or **restore freshness** |

**Real-life analogy:** DORA for data is not “how fast we edit SQL.” It is **how often we safely change the factory line** without shipping defective batches.

---

## Deployment frequency — shipping pipeline change safely

**High deployment frequency** for data teams does **not** mean “run full refresh hourly because we can.”

It means:

- **Small, incremental** dbt model changes merged often
- **Feature flags** for new marts (`_v2` tables, shadow pipelines)
- **CI** that runs tests on every PR
- **Idempotent** jobs — safe to re-run

### What counts as a “deploy”

Be consistent in your metrics:

- dbt model/tag deploy to prod target
- Airflow DAG bundle promotion
- Spark JAR + config rollout
- Streaming job savepoint redeploy

### Junior → senior shift

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Big bang migration Friday night.” | **Strangler pattern** — parallel tables, cutover with checks |
| “Deploy = run job manually.” | Deploy = **versioned artifact** + audit trail |
| “More runs = more progress.” | More **validated** merges with observability |

**Healthy pattern:** Frequency ↑ with **stable CFR** and **stable freshness SLO**.

**Unhealthy pattern:** Frequency ↑, **dashboards wrong half the week** — you are not elite, you are **roulette**.

---

## Change failure rate — when a merge hurts consumers

**CFR for data** should include failures that **matter downstream**, not only task exceptions:

| Counts toward CFR | Often excluded (team policy) |
|-------------------|------------------------------|
| Mart published with failed **dbt test** | Dev environment failure |
| **Silent schema break** (column gone) | Skipped optional staging model |
| Bad partition requiring **backfill** | Retry succeeded in < 5 min with no consumer impact |
| Wrong currency / duplicate keys in prod | Experimental sandbox table |

Rough form (same as Part 1):

$$
\text{CFR} = \frac{\text{failed or harmful pipeline changes}}{\text{total pipeline changes}} \times 100\%
$$

**Real-life analogy:** A food plant tracks **recalls**, not “conveyor belt stopped for 30 seconds with no bad shipment.”

### Reducing CFR without freezing forever

- **Contract tests** on upstream APIs and landing zones
- **Impact analysis** — which dashboards break if this column moves?
- **Canary datasets** — publish to `_beta` schema first
- **Row-level comparisons** old vs new before swap

---

## Lead time and MTTR — recovery in data land

### Lead time for changes

Measure: **merge to prod truth**.

Includes: CI, review queue, deployment window, **backfill duration** if historical recompute is required.

**Senior pain point:** A “one-line fix” that requires **30-day backfill** is not a small change — say so in planning.

### MTTR — Mean Time to Recovery

For pipelines, recovery might mean:

- **Rerun** from failed task with correct params
- **Point-in-time backfill** for bad partitions
- **Rollback** dbt version + invalidate caches
- **Communicate** “do not use table X until 14:00”

**MTTR clock starts** when consumers **could** be harmed — not when you noticed a red square for fun.

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Re-run fixed it.” | “How long were **wrong numbers** visible? Who was notified?” |
| “Root cause next week.” | **Mitigate in minutes** — disable mart, serve yesterday, page comms |
| “Backfill is manual heroics.” | **Playbooks** + idempotent jobs + partition-level reruns |

**Real-life analogy:** Restaurant serves undercooked chicken (incident). MTTR is not “chef learned why” — it is **how fast plates stop leaving the kitchen wrong** and **customers get told**.

### Streaming note

For Kafka/Flink paths, MTTR often pairs with **lag SLO** and **duplicate handling**. Recovery = lag back under threshold **and** correctness checks green.

---

## MTBF — pipelines that break every Tuesday

**MTBF** asks: how long between **meaningful** failures?

Chronic patterns:

- Same **upstream file** late every Monday
- **OOM** on day 31 of every month (partition explosion)
- **Schema drift** after vendor “silent” column add
- **Race** between two DAGs writing same table

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “We re-run on failure.” | “Why is failure **scheduled**?” |
| “Flaky is normal.” | Flakiness is **MTBF debt** — automate detection |
| “Not my DAG’s fault.” | **End-to-end ownership** of the promise to consumers |

**Senior move:** Track **repeat incidents** in postmortems — if the same root cause appears twice, the fix was cosmetic.

**Real-life analogy:** If your car needs a jump start **every Monday**, the problem is not “you forgot cables” — it is the **alternator**.

---

## Metrics dashboards data seniors actually watch

One screen is enough if it answers **“are we trustworthy right now?”**

| Metric | Why it matters |
|--------|----------------|
| **Freshness lag** per critical mart | Consumer-facing pain |
| **SLO compliance %** (rolling 28d) | Error budget health |
| **Job success rate** (with retry stripped) | Hidden flakiness |
| **dbt test / DQ pass rate** | Correctness before publish |
| **CFR** (pipeline changes) | Delivery discipline |
| **MTTR** (last 3 incidents) | Recovery muscle |
| **Cost per TB / per pipeline** | ROI conversations |
| **Backlog depth** (queued runs) | Utilization / capacity |

**Utilization warning (from Part 1):** Cluster at 95% nightly sounds efficient until **one 20% larger source file** delays finance close. Headroom is **shock absorption**.

**Real-life analogy:** Air traffic control does not only watch “planes took off.” They watch **delays, near-misses, and runway capacity** together.

---

## Performance reviews — prove impact without ticket spam

Managers are not impressed by **47 JIRA tickets closed**. They remember **trust, risk removed, and money/time saved**.

### Frame outcomes, not activity

| Weak bullet | Strong bullet |
|-------------|---------------|
| “Maintained Airflow DAGs.” | “Kept **revenue mart freshness SLO at 99.4%** (target 99%) while reducing run cost **12%**.” |
| “Fixed pipeline bugs.” | “Cut **MTTR** for tier-1 marts from 4h → 45m with partition backfill playbook.” |
| “Added dbt tests.” | “Reduced **bad publishes (CFR)** from 8% → 2% over Q2 via contract tests on landing zone.” |
| “Worked on migration.” | “Delivered migration with **zero SLA breaches**; burned **22% error budget** — documented in postmortem.” |

### Use the STAR pattern with metrics

- **Situation:** “Finance close depends on mart X by 06:00.”
- **Task:** “SLO breaching 2×/week; root cause unclear.”
- **Action:** “Partition-level freshness alerts, upstream SLA doc, idempotent backfill.”
- **Result:** “**99.2% on-time** over 90 days; **one** breach vs nine prior quarter.”

### Talk about trade-offs (this sounds senior)

> “I chose to **miss freshness by 2 hours** once rather than publish **unreconciled** Stripe data — preserved correctness, spent error budget, notified stakeholders.”

That shows **judgment**, not heroics.

### What to bring to the review (checklist)

- 1–2 **SLO charts** (before/after)
- **One incident** where you improved MTTR or MTBF
- **One delivery win** (lead time or CFR)
- **One business line** — analyst hours saved, infra cost, decision delay avoided
- **One thing you would do next quarter** tied to error budget policy

**Real-life analogy:** Performance review is a **investor pitch** for your future salary — show **returns**, not keystrokes.

---

## A quarter plan template for pipeline owners

Copy into your notes and fill honestly:

| Month | Focus | Metric target | Stop doing if red |
|-------|-------|---------------|-------------------|
| M1 | Stabilize freshness on mart A | SLO ≥ 99% | Optional refactors |
| M2 | Lower CFR on dbt deploys | CFR < 5% | Large schema moves |
| M3 | Reduce MTTR | P95 < 1h | New feature marts |

**Weekly habit (15 min):**

1. Check **error budget** on tier-1 SLIs  
2. Check **CFR** on last 5 prod changes  
3. Any **repeat** failure pattern? (MTBF)  
4. One **consumer** check-in — do numbers match their reality?

---

## Conclusion — the senior data engineer sentence

Junior you asked: **“Did the job succeed?”**  
Senior you asks: **“Did we keep our data promise — and did we spend our error budget wisely?”**

For **data pipelines**, that means:

- **SLOs** consumers can repeat in a meeting  
- **Error budgets** that change when you ship risky work  
- **DORA-style discipline** on how often you change prod and how often it hurts  
- **MTTR / MTBF** that reflect **bad data exposure**, not only red tasks  
- **Performance reviews** that cite **outcomes**, not ticket counts  

You do not need a perfect data mesh slide deck. You need **one critical mart** with a real SLO, a tracked budget, and a story you can tell without apologizing.

**Part 1 recap:** [Junior → senior metrics (general)](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md)

**Feedback welcome:** if you want Part 3 on **streaming-specific SLOs** (lag, watermark, exactly-once) or **data contracts with upstream teams**, say which stack you run — Kafka, Spark Structured Streaming, Flink, or managed warehouse pipelines.
