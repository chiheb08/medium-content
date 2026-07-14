# The Pipeline Promised Fresh Data by 8 a.m. — Error Budgets, DORA, and Reviews for Data Teams

**Part 2 of:** *Stop Optimizing Your Ticket Count: The Metrics That Change When You Go From Junior to Senior*

**Alternative titles for Medium:**
- *Error Budgets for Data Pipelines — A Practical Follow-Up for Data Engineers*
- *DORA Metrics When Your “Deploy” Is a DAG Run*
- *How to Talk About Pipeline Reliability in Your Performance Review*

**Subtitle:** Freshness SLOs, job failure rates, recovery time, and the language that makes data engineering look senior — not just busy.

> **Medium tip:** Paste section headings as **H2** blocks. Use the **“At a glance”** list as your scroll map. For tables, screenshot from the full `.md` file or use the `*-medium-no-tables.md` companion.

---
> **Medium paste version:** Empty slots for tables. Screenshot from full `.md` file.


## At a glance — what this follow-up gives you

- **Error budgets in practice** for data pipelines — freshness, completeness, and when to ship risky changes
- **DORA metrics translated** for data teams (deployment frequency = pipeline releases; CFR = failed runs / bad publishes)
- **Pipeline-native reliability:** MTTR for backfills, MTBF for chronic flakiness, late data, schema drift
- **How to present impact** in performance reviews without listing every ticket
- **Junior vs senior** habits specific to batch, streaming, and hybrid stacks

**One line to remember:** A green Airflow task is not a promise — an **SLO with an error budget** is.

**Who this is for:** data engineers, analytics engineers, and platform folks who own **pipelines, warehouses, and SLAs to downstream teams** — not generic product on-call (though the ideas rhyme).

> **How this article reads:** Look for **📋 Work story** boxes — short scenes from real data jobs (finance dashboards, late upstream files, bad dbt deploys) explained in plain language.

> **New to the jargon?** Read **[Terms defined — data pipeline dictionary](#terms-defined--data-pipeline-dictionary)** first.

---

## Terms defined — data pipeline dictionary

Terms that confuse **non-data** folks — or juniors hearing them for the first time.

### Core ideas


> **📷 Insert table screenshot here (Table 1)**
> *Core ideas*

<!-- Empty slot -->


### Tools you will hear (not required to master every one)


> **📷 Insert table screenshot here (Table 2)**
> *Tools you will hear (not required to master every one)*

<!-- Empty slot -->


### Data quality and shape


> **📷 Insert table screenshot here (Table 3)**
> *Data quality and shape*

<!-- Empty slot -->


### Metrics from Part 1 (quick reminder)


> **📷 Insert table screenshot here (Table 4)**
> *Metrics from Part 1 (quick reminder)*

<!-- Empty slot -->


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

### 📋 Work story (simple) — the scene everyone knows

**7:55 a.m.** — Finance opens the CEO revenue dashboard.  
**7:56 a.m.** — Slack: “Why does yesterday show **$0**?”  
You check Airflow: **all green** ✅  
You check the mart: **empty partition** for yesterday because upstream landed late and the job “succeeded” with **zero rows**.

**Junior:** “But the task didn’t fail.”  
**Senior:** “The **promise** failed — and Finance does not care about our task color.”

---

Every pipeline makes an implicit promise. Seniors make it **explicit**.


> **📷 Insert table screenshot here (Table 5)**
> *Table*

<!-- Empty slot -->


**Pitfall:** Teams optimize **job success rate** while **freshness SLO** is on fire — like celebrating an on-time flight that landed in the wrong city.

### 📋 Work story (simple) — five promises, one angry PM


> **📷 Insert table screenshot here (Table 6)**
> *📋 Work story (simple) — five promises, one angry PM*

<!-- Empty slot -->


**Real-life analogy:** A restaurant promises **hot food, correct bill, clean table**. The kitchen timer beeping only covers **one** of those.

---

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

### 📋 Work story (simple)

You promise Product: **“Sales mart ready by 6 a.m.”**  
That is the **SLO**.  
Legal’s contract with a partner might say **“8 a.m. or penalty”** — that is the **SLA**.  
You aim for **6** so missing by a bit still beats **8**.

**Everyday analogy:** You set your alarm **30 minutes early** so traffic does not make you late for the interview.

---


> **📷 Insert table screenshot here (Table 7)**
> *Table*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 8)**
> *Pipeline-specific budget spenders*

<!-- Empty slot -->


**Senior move:** Log **why** budget burned in the incident channel — “late because Stripe export slipped” vs “late because we had no alert.”

### 📋 Work story (simple) — spending budget on purpose

Black Friday week. Marketing runs a **risky** catalog sync. It might slip freshness by **2 hours** — but revenue depends on it.  
**Senior:** “We have **40% error budget** left. This spend is **worth it** — I’ll post in #data-incidents and watch the partition alerts.”

Same week, someone wants a **cosmetic refactor** of a stable mart.  
**Senior:** “Budget is thin — defer to January.”

**Everyday analogy:** You save vacation days for **family wedding**, not for **skipping work when you’re bored**.

---

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


> **📷 Insert table screenshot here (Table 9)**
> *Red zone — budget exhausted or SLA at risk*

<!-- Empty slot -->


**Real-life analogy:** Hospital ER — when beds are full, **elective surgeries pause**. Not because surgery is bad, because **trust and capacity** are finite.

### The conversation that sounds senior in a meeting

> “We have **18% error budget left** on revenue mart freshness. I recommend we **defer** the dimension table refactor until next sprint and ship only the **null-rate fix** that stops bad joins.”

That is not bureaucracy. That is **risk management in plain language**.

### 📋 Work story (simple) — saying no with numbers

Your PM wants both **new metrics** and **zero risk** this week.  
**Junior:** silent stress + weekend work.  
**Senior:** “Freshness SLO has **12% budget** left. Pick **one** schema change; the other waits. Here’s the chart.”

Numbers turn arguments into **choices**.

---

[DORA metrics](https://dora.dev/) were framed for **application delivery**. Data platforms map cleanly if you translate nouns:


> **📷 Insert table screenshot here (Table 10)**
> *Table*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 11)**
> *Junior → senior shift*

<!-- Empty slot -->


**Healthy pattern:** Frequency ↑ with **stable CFR** and **stable freshness SLO**.

**Unhealthy pattern:** Frequency ↑, **dashboards wrong half the week** — you are not elite, you are **roulette**.

### 📋 Work story (simple) — small merges beat Friday night

**Old way:** one **big** migration every 6 weeks — Friday 11 p.m., pizza, prayer.  
**New way:** ship **one dbt model** per day with tests; shadow table `_v2` for a week, then swap.

Deploy frequency goes **up**. Bad surprises go **down**.

**Everyday analogy:** Packing for a trip **a little each night** beats one panic suitcase at 4 a.m.

---

**CFR for data** should include failures that **matter downstream**, not only task exceptions:


> **📷 Insert table screenshot here (Table 12)**
> *Table*

<!-- Empty slot -->


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

### 📋 Work story (simple) — the dbt deploy that “succeeded”

You merge a dbt model rename: `revenue` → `total_revenue`.  
dbt run: **green**.  
Looker dashboard: **broken** all morning because nobody updated the explore.  
Finance used **cached wrong** numbers until 2 p.m.

**CFR event?** Yes — if your policy says **consumer harm** counts.  
**Lesson:** deploy includes **downstream comms**, not only SQL.

**Everyday analogy:** You changed the **street name** but did not update **Google Maps**.

---

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


> **📷 Insert table screenshot here (Table 13)**
> *Table*

<!-- Empty slot -->


**Real-life analogy:** Restaurant serves undercooked chicken (incident). MTTR is not “chef learned why” — it is **how fast plates stop leaving the kitchen wrong** and **customers get told**.

### 📋 Work story (simple) — wrong numbers in prod

**9:00** — bad join doubles revenue in the mart.  
**9:05** — senior disables the mart view, pins message in Slack: **“Use yesterday until 11:00.”**  
**10:30** — partition backfill completes, tests pass, view re-enabled.

**Wrong data exposure:** ~1.5 hours — that is what **MTTR** measures, not “when dbt turned green again.”

---

For Kafka/Flink paths, MTTR often pairs with **lag SLO** and **duplicate handling**. Recovery = lag back under threshold **and** correctness checks green.

---

## MTBF — pipelines that break every Tuesday

**MTBF** asks: how long between **meaningful** failures?

Chronic patterns:

- Same **upstream file** late every Monday
- **OOM** on day 31 of every month (partition explosion)
- **Schema drift** after vendor “silent” column add
- **Race** between two DAGs writing same table


> **📷 Insert table screenshot here (Table 14)**
> *Table*

<!-- Empty slot -->


**Senior move:** Track **repeat incidents** in postmortems — if the same root cause appears twice, the fix was cosmetic.

**Real-life analogy:** If your car needs a jump start **every Monday**, the problem is not “you forgot cables” — it is the **alternator**.

### 📋 Work story (simple) — the file that is always late

Every Monday, the **vendor CSV** lands at 10:00 instead of 6:00. Someone re-runs the DAG. Green by noon.  
**Six months** of this — nobody asks **why Monday**.

Senior move: alert on **upstream SLA**, not only your task. Fix the **vendor contract** or **buffer the SLO**.

**Everyday analogy:** Your bus is **late every Monday** — buying a faster alarm clock does not fix the bus.

---

## Metrics dashboards data seniors actually watch

One screen is enough if it answers **“are we trustworthy right now?”**


> **📷 Insert table screenshot here (Table 15)**
> *Metrics dashboards data seniors actually watch*

<!-- Empty slot -->


**Utilization warning (from Part 1):** Cluster at 95% nightly sounds efficient until **one 20% larger source file** delays finance close. Headroom is **shock absorption**.

**Real-life analogy:** Air traffic control does not only watch “planes took off.” They watch **delays, near-misses, and runway capacity** together.

### 📋 Work story (simple) — one screen in standup

Monday standup: someone shares **one** dashboard — freshness lag, SLO %, CFR last week.  
**No** scrolling through 40 Grafana tabs.  
When finance asks “can we trust revenue?”, the answer is **one number**: **99.2% on-time, budget green**.

**Everyday analogy:** Doctor checks **blood pressure + heart rate + temperature** — not 50 lab reports every visit.

---

Managers are not impressed by **47 JIRA tickets closed**. They remember **trust, risk removed, and money/time saved**.

### 📋 Work story (simple) — what your manager remembers

**Engineer A:** “Closed 52 tickets, maintained DAGs.”  
**Engineer B:** “Revenue mart **99.4% on-time** (target 99%). Cut bad publishes from **8% → 2%**. Saved **~$4k/month** cluster cost.”

Both worked hard. Only one story survives **budget season**.

---

### Frame outcomes, not activity


> **📷 Insert table screenshot here (Table 16)**
> *Frame outcomes, not activity*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 17)**
> *A quarter plan template for pipeline owners*

<!-- Empty slot -->


**Weekly habit (15 min):**

1. Check **error budget** on tier-1 SLIs  
2. Check **CFR** on last 5 prod changes  
3. Any **repeat** failure pattern? (MTBF)  
4. One **consumer** check-in — do numbers match their reality?

### 📋 Work story (simple) — 15 minutes every Friday

Friday **3:00 p.m.**, calendar block: **pipeline health**.  
Open one page: SLO %, CFR, any repeat incident.  
Takes **15 minutes** — catches the “always late Monday file” before it becomes **quarterly fire drill**.

**Everyday analogy:** Weekly **weigh-in** on a diet — small check beats discovering in June you never lost the weight.

---

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
