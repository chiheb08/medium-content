# Your Stream Is “Green” but Finance Is Wrong — Streaming SLOs, Lag, and Senior Thinking (Part 3)

**Part 3 of:** *Stop Optimizing Your Ticket Count* → [Part 1 (general)](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md) · [Part 2 (batch pipelines)](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/data-pipelines-error-budgets-dora-performance.md)

**Alternative titles for Medium:**
- *Lag Is Not a Number — Streaming Metrics for Engineers Who Want to Sound Senior*
- *Event Time, Watermarks, and Error Budgets: Part 3 for Kafka, Flink, and Spark Streaming Teams*
- *Exactly-Once Is a Story You Tell Stakeholders — Streaming Reliability in Plain Language*

**Subtitle:** Consumer lag, end-to-end latency, watermarks, delivery semantics, and how to present streaming impact — without pretending the tutorial cluster is production.

> **Medium tip:** Paste section headings as **H2** blocks. For tables, use `streaming-error-budgets-slo-performance-medium-no-tables.md` or screenshot from the full article.

---

## At a glance — what Part 3 gives you

- **Streaming-native SLOs:** lag, freshness, end-to-end latency, completeness — and what *not* to promise
- **Event time vs processing time** and **watermarks** — why “real-time” still lies
- **Delivery semantics** (at-most / at-least / exactly-once) in **honest** language for stakeholders
- **Error budgets** for always-on systems — burn, freeze, and replay policy
- **DORA & recovery** when your “deploy” is a savepoint, consumer group, or schema bump
- **Performance review bullets** that prove you own **continuous** data, not one nightly job

**One line to remember:** In streaming, junior you watches **consumer lag**. Senior you watches **correctness under delay, duplication, and deploy**.

**Stacks referenced:** Kafka-style logs, Spark Structured Streaming, Flink — patterns apply to Kinesis, Pulsar, Redpanda, and managed stream processors too.

> **New to the jargon?** Read **[Terms defined — streaming dictionary](#terms-defined--streaming-dictionary)** first. For SLO, error budget, MTTR, DORA, see [Part 1](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md#terms-defined--the-dictionary-read-this-first). For batch pipelines, see [Part 2’s dictionary](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/data-pipelines-error-budgets-dora-performance.md#terms-defined--data-pipeline-dictionary).

---

## Terms defined — streaming dictionary

### Streaming in one sentence

**Streaming** = data is processed **continuously** as events arrive, instead of waiting for a nightly batch job to finish.

### Core streaming concepts

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Event** | One **record** of something that happened (click, payment, sensor reading) | The atom of streaming — usually immutable once written. |
| **Stream** | An **unending sequence** of events | The job never “completes” like a daily batch. |
| **Broker / log** | System that **stores and forwards** events (Kafka, Pulsar, Kinesis) | Durable mailbox between producers and consumers. |
| **Topic** | Named **channel** of events (like a TV channel for data) | Consumers subscribe to topics they care about. |
| **Producer** | System that **writes** events into the stream | Upstream of your pipeline. |
| **Consumer** | System that **reads** events and processes them | Your Flink/Spark job or microservice. |
| **Consumer group** | Team of consumers **sharing** the work of reading a topic | Enables scale — each partition read by one worker in the group. |
| **Partition** | **Shard** of a topic — events with same key go to same partition | Enables parallelism; **skew** happens when one partition is huge. |
| **Offset** | **Bookmark** — “how far we have read” in the log | Resetting offset = **replay** or **skip** data — dangerous without a runbook. |

### Time and correctness

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Event time** | When the thing **actually happened** in the real world | Correct for analytics when mobile/offline delay exists. |
| **Processing time** | When **your pipeline processed** the event | Simpler but **misleading** under delay. |
| **Watermark** | Engine’s estimate: “we are **unlikely** to see events older than T” | Used to **close windows** and emit results. |
| **Window** | Grouping events by **time bucket** (e.g. “counts per 5 minutes”) | Aggregates need boundaries — windows define them. |
| **Allowed lateness** | Policy: “still accept late events up to **N minutes**” | Trade-off between **completeness** and **speed**. |
| **End-to-end latency** | Time from **event happens** → **visible downstream** | What fraud, ops, and product actually feel. |
| **Consumer lag** | How **far behind** the consumer is vs newest event in the log | Famous metric — **not sufficient alone** for correctness. |

### Delivery and failure words

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **At-most-once** | Each event processed **0 or 1 times** — may **lose** data | Fast; risky for money metrics. |
| **At-least-once** | No loss; **retries** may create **duplicates** | Common; needs **idempotent** sinks. |
| **Exactly-once** | Each event affects downstream **once** (as defined) | Marketing-heavy — usually **end-to-end engineering**, not one config flag. |
| **Idempotency** | Doing the same operation **twice** has same effect as once | e.g. upsert by `event_id` — prevents double-counting. |
| **Checkpoint (Flink)** | **Snapshot** of job state for fault tolerance | Slow checkpoints → lag and instability. |
| **Savepoint** | **Manual checkpoint** for controlled upgrade/rollback | Streaming “deploy” best friend. |
| **Replay** | **Re-read** events from earlier offset | Fixes bugs; can **duplicate** if sink not idempotent. |
| **Backpressure** | Downstream slow → upstream **slows or queues** | Warning sign before lag explodes. |
| **Skew / hot key** | One key has **most** traffic — one worker overloaded | p99 latency dies while average looks fine. |

### Schema and deploy

| Term | Plain English | Why it matters |
|------|---------------|----------------|
| **Schema registry** | Central store for **event structure** (Avro, Protobuf) | Prevents “field disappeared” surprises. |
| **Compatibility (BACKWARD/FORWARD)** | Rules for **safe** schema evolution | Breaking change = silent **CFR** event. |
| **Sink** | Where results **go** (warehouse, DB, cache) | Often the real **latency** bottleneck. |
| **Source** | Where events **come from** | Lag starts here. |

### Metrics reminders (Part 1 & 2)

| Term | Streaming twist |
|------|-----------------|
| **SLO** | Often on **lag** or **latency p99**, not “job green” |
| **Error budget** | **Minutes per month** over lag threshold |
| **MTTR** | Includes **replay, savepoint rollback, reconciliation** |
| **MTBF** | “Breaks every sale day” = chronic design debt |
| **CFR** | Bad deploy = wrong live aggregates, not only job crash |

---

## Table of contents

1. [Terms defined — streaming dictionary](#terms-defined--streaming-dictionary)
2. [Why streaming needs Part 3 — the green dashboard trap](#why-streaming-needs-part-3--the-green-dashboard-trap)
2. [Why streaming needs Part 3 — the green dashboard trap](#why-streaming-needs-part-3--the-green-dashboard-trap)
3. [Batch vs streaming promises — different scoreboards](#batch-vs-streaming-promises--different-scoreboards)
4. [The streaming SLI menu — what to measure](#the-streaming-sli-menu--what-to-measure)
5. [Consumer lag — the metric everyone cites (and misreads)](#consumer-lag--the-metric-everyone-cites-and-misreads)
6. [End-to-end latency — when the event actually matters](#end-to-end-latency--when-the-event-actually-matters)
7. [Event time, processing time, and watermarks](#event-time-processing-time-and-watermarks)
8. [Completeness and correctness under delay](#completeness-and-correctness-under-delay)
9. [Delivery semantics — what you can actually promise](#delivery-semantics--what-you-can-actually-promise)
10. [Error budgets for always-on pipelines](#error-budgets-for-always-on-pipelines)
11. [DORA when deploy means savepoint](#dora-when-deploy-means-savepoint)
12. [MTTR and MTBF in stream land](#mttr-and-mtbf-in-stream-land)
13. [Backpressure, skew, and utilization](#backpressure-skew-and-utilization)
14. [Schema change and compatibility — silent CFR multipliers](#schema-change-and-compatibility--silent-cfr-multipliers)
15. [The senior streaming dashboard](#the-senior-streaming-dashboard)
16. [Performance reviews for streaming engineers](#performance-reviews-for-streaming-engineers)
17. [Quarter plan — streaming edition](#quarter-plan--streaming-edition)
18. [Conclusion — the senior streaming sentence](#conclusion--the-senior-streaming-sentence)

---

## Why streaming needs Part 3 — the green dashboard trap

[Part 2](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/data-pipelines-error-budgets-dora-performance.md) treated pipelines that **finish**: daily marts, Airflow success, freshness by 8 a.m.

**Streaming never finishes.** The job is always running. The tutorial said “events flow in real time.” Production adds:

- retries and **duplicates**,
- **late** events (phone offline, CDC backlog),
- **skew** (one hot partition),
- **replay storms** after outages,
- **schema** changes from teams you do not control.

**Junior streaming instinct:** “Lag is low and the Flink job is RUNNING.”  
**Senior streaming instinct:** “Under load, delay, and failure — are **downstream decisions** still **correct enough**, **on time enough**, and **explainable**?”

**Real-life analogy:** A **live TV broadcast** (streaming) vs **next-day replay** (batch). The director cares if viewers see the goal **now** — and that the score graphic is **not wrong** because a delayed packet duplicated a point.

---

## Batch vs streaming promises — different scoreboards

| Dimension | Batch (Part 2) | Streaming (Part 3) |
|-----------|----------------|---------------------|
| **Unit of success** | Job run succeeded | Pipeline **healthy over time** |
| **Freshness** | “Table ready by 06:00” | “Event visible within **N seconds/minutes**” |
| **Failure signal** | Red task | Lag spike, dropped records, **wrong** aggregates |
| **Recovery** | Rerun / backfill partition | Offset reset, savepoint, replay, **restate** |
| **Deploy risk** | Bad SQL once | Bad state **forever** until noticed |

**Senior move:** Do not copy batch SLO slides and replace “daily” with “real-time.” Define **which event**, **which path**, **which latency percentile**.

---

## The streaming SLI menu — what to measure

Pick **one primary SLI per consumer-facing stream** — not twelve graphs nobody owns.

| SLI | Definition (typical) | Consumer cares because… |
|-----|----------------------|-------------------------|
| **Consumer lag** | Offset behind log end (records or time) | Data is **stale** |
| **End-to-end latency** | `processing_time - event_time` (p95/p99) | Decisions are **late** |
| **Freshness of sink** | Age of latest row in serving store | Dashboards **lie by age** |
| **Duplicate rate** | Extra records vs idempotent expectation | Metrics **double-count** |
| **Loss rate** | Missing events vs source (hard — needs audit) | Revenue / risk **under-reported** |
| **Availability** | Job up % / no critical alert firing | **Silence** = blind spot |

**Pitfall:** Optimizing **average** lag while **p99** kills fraud detection or inventory.

---

## Consumer lag — the metric everyone cites (and misreads)

**Consumer lag** = how far a consumer group is behind the **end of the log** (Kafka: `records-lag-max`; Flink: source lag + checkpoint health).

**Defined in plain English:** Imagine a **conveyor belt** of events. Lag is “how many packages are still **in front of you** before you are caught up to the newest one.” Low lag = you are keeping pace **right now**.

### What lag actually means

- **Low lag** → you are **keeping up** at this moment.
- **Low lag does not mean** → correct joins, no duplicates, no late drops.

**Real-life analogy:** You are **reading the newspaper** only one page behind the printer — but you might be **reading the wrong edition** if sorting was wrong.

### Junior → senior shift

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Lag < 1000 messages = healthy.” | “Lag in **what unit**? Under **what traffic**? For **which** consumer group?” |
| “One lag number for the cluster.” | **Per-partition** lag — hot keys hide in averages |
| “Alert on any lag.” | Alert on **SLO breach** + **growth rate**, not noise |

### Lag SLO example

- **SLO:** p99 consumer lag **≤ 60 seconds** during business hours, 28-day rolling window.
- **Error budget:** allowed minutes per month above threshold.

**Stack notes:**

- **Kafka:** watch `records-lag-max`, `records-lag-avg`, and **time-based lag** if using timestamps in metrics.
- **Flink:** source idle time, checkpoint duration, **backpressure** ratio — lag symptoms often start there.
- **Spark Structured Streaming:** `maxOffsetsBehindLatest` or processing delay — know your trigger interval.

---

## End-to-end latency — when the event actually matters

**End-to-end latency** tracks the path:

> event happens in the world → appears in the **decision surface** (feature store, dashboard, API).

Often:

$$
\text{latency} = t_{\text{visible downstream}} - t_{\text{event}}
$$

*In plain English:* how old is the fact when someone acts on it?

### Why this beats “job CPU is fine”

A Flink job can run at 40% CPU while latency drifts because:

- **checkpoint** duration grew,
- **sink** throttles (warehouse ingest limits),
- **micro-batch** interval increased,
- **GC** pauses on state-heavy operators.

**Real-life analogy:** Package **scanned at warehouse** (processor healthy) but **not on your doorstep** (sink SLO) — you care about doorstep time.

### SLO framing

| Use case | Example latency SLO |
|----------|---------------------|
| Ops dashboard | p95 < 30s |
| Fraud scoring | p99 < 500ms |
| Analytics rollups | p95 < 5 min |
| ML features | < 1 min for 99% of users |

**Senior question:** “Latency SLO for **whom** — and what happens when we miss: block, degrade, or alert only?”

---

## Event time, processing time, and watermarks

### Two clocks

| Clock | Meaning | Analogy |
|-------|---------|---------|
| **Event time** | When it **happened** in the real world | When the goal was scored |
| **Processing time** | When your **pipeline saw** it | When your TV received the frame |

They diverge — always.

### Watermarks (plain language)

A **watermark** is the engine’s bet:

> “We are unlikely to see events much older than **T** now.”

Used to **close windows** and emit aggregates.

**Real-life analogy:** A sports broadcast says **“results are final through minute 80”** — late VAR reviews after that need a **correction policy**, not silent rewrites.

### Junior → senior shift

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Use processing time — simpler.” | **Event time** for correctness when late data is real |
| “Watermark = magic truth.” | Watermark = **heuristic** — document allowed lateness |
| “Late events are bugs.” | Late events are **physics** — phones, retries, CDC |

### Allowed lateness (Flink / Structured Streaming)

Explicit rule: “Accept updates up to **10 minutes** late; after that, side output or drop with metric.”

**Senior move:** Metric **dropped / late records** on the dashboard — not only lag.

---

## Completeness and correctness under delay

Streaming systems often choose **approximate now** vs **correct later**.

| Strategy | Trade-off |
|----------|-----------|
| **Emit provisional counts** | Fast, may revise |
| **Hold window until watermark** | Slower, more complete |
| **Lambda / batch correction** | Stream for speed, batch for truth |

**Completeness SLI (example):** reconciled stream totals within **0.1%** of batch truth daily.

**Real-life analogy:** **Election night projections** vs **official count** — both useful; only one should be in the legal filing.

**Senior sentence for stakeholders:**

> “We optimize for **p99 latency under 1 minute** with **end-of-day reconciliation** to batch; intraday numbers are **directional**.”

That is honesty — not weakness.

---

## Delivery semantics — what you can actually promise

Marketing loves **exactly-once**. Seniors translate:

| Guarantee | Plain English | Typical mechanism |
|-----------|---------------|-------------------|
| **At-most-once** | May **lose** events; no duplicates | Fire-and-forget |
| **At-least-once** | **No loss**; may **duplicate** | Retries + idempotent sinks |
| **Exactly-once** (end-to-end) | Neither loss nor dup **as observed by sink** | Transactions, idempotent keys, dedup store |

**Truth:** End-to-end exactly-once is **engineering plus business definition**, not a checkbox on a Flink slide.

**Real-life analogy:** **Bank transfer:**

- At-most-once: money might **vanish** from sender without arriving.
- At-least-once: might **charge twice** without idempotency keys.
- Exactly-once: **one** visible transfer — what users care about.

### What to document

- **Idempotency key** on sink (e.g. `event_id`)
- **Dedup** window length
- **Side effects** outside DB (email, webhook) — often **at-least-once** forever

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Enable exactly-once in config.” | “Show me **end-to-end** proof under **failure**.” |
| “Duplicates are impossible now.” | “Duplicates are **rare** — metric + reconciliation.” |
| “Sink failed once — we’re fine.” | “Did retry **double** revenue in the mart?” |

---

## Error budgets for always-on pipelines

Batch budgets often count **missed days**. Streaming budgets count **bad minutes** or **breach seconds**.

### Example lag budget

- **SLO:** p99 lag ≤ 120s for 99.9% of business hours per month.
- **Budget:** ~0.1% of business minutes ≈ **~26 minutes/month** (order-of-magnitude; calibrate to your calendar).

### What burns streaming budget

| Event | Budget impact |
|-------|----------------|
| Planned savepoint deploy with 5 min lag spike | **Planned** — communicate |
| Hot partition during sale | **Capacity** — may be acceptable if SLO holds |
| Bad state schema deploy | **CFR** — unplanned, fix MTBF |
| Silent drop of late events | **Correctness** — worse than lag |

### Green / yellow / red (streaming)

| Zone | Signal | Action |
|------|--------|--------|
| **Green** | Lag SLO met; reconciliation clean | Normal deploys; tune cost |
| **Yellow** | Lag p99 rising; checkpoint slow | Freeze state schema changes; scale; profile |
| **Red** | SLO breach + consumer impact | Change freeze; replay playbook; comms |

**Real-life analogy:** **Air traffic** — minor delays OK within buffer; **runway closure** stops departures.

---

## DORA when deploy means savepoint

Map DORA to streaming releases:

| DORA | Streaming translation |
|------|------------------------|
| **Deployment frequency** | Job redeploys, savepoint restores, consumer group resets (controlled), schema registry v2 |
| **Lead time** | PR merge → prod job reflecting change |
| **CFR** | Deploys causing **lag SLO breach**, **wrong aggregates**, or **mandatory replay** |
| **MTTR** | Time to **restore SLO** after bad release |

### Safer streaming deploy patterns

- **Savepoint** before stateful change (Flink)
- **Blue/green** consumer on new topic version
- **Canary** partition or shadow sink
- **Compatible** schema changes (Avro/Protobuf rules)

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Stop job, edit code, start job.” | **Savepoint**, **drain**, **validate** lag + correctness |
| “Reset offsets to fix.” | Offset reset = **data loss or duplication** — runbook + comms |
| “Deploy Friday 5 p.m.” | Deploy when **rollback** and **on-call** match risk |

---

## MTTR and MTBF in stream land

### MTTR — restore trust, not only green status

Clock starts when **consumers** or **SLOs** hurt — not when you noticed a yellow widget.

**Recovery playbook (examples):**

1. **Scale** parallelism / replicas  
2. **Throttle** upstream or shed load  
3. **Rollback** to savepoint  
4. **Replay** from known-good offset with idempotent sink  
5. **Fail over** to degraded read model  
6. **Notify** downstream “do not automate trades on stream X”  

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Restart fixed it.” | “How many **bad events** were written? **Backfill**?” |
| “Lag zero — close incident.” | “Reconcile **counts** with source of truth.” |

### MTBF — chronic streaming pain

Repeat offenders:

- Sale day lag **every quarter**
- Checkpoint timeout **every** state size threshold
- **OOM** when one key dominates
- Consumer **rebalance storm** on every deploy

**Senior move:** Same root cause twice → **design** fix (salting keys, incremental snapshots, separate hot/cold paths).

**Real-life analogy:** Power flickers **every storm** — buying a new lamp each time is not MTTR excellence.

---

## Backpressure, skew, and utilization

### Backpressure

When downstream cannot keep up, pressure travels **upstream** — slower consumption, growing lag, possible **data loss** if buffers overflow.

**Senior signal:** Flink **backpressure** web UI; Kafka **consumer fetch rate** vs produce; Spark **scheduling delay**.

**Real-life analogy:** Highway on-ramp **metering** — not cowardice, **stability**.

### Skew

One partition carries 40% of keys → one task holds the whole pipeline hostage.

**Mitigations:** key salting, custom partitioner, **separate** hot path.

### Utilization (Part 1 callback)

100% CPU on stream workers = **no burst headroom**. Seniors leave room for **replay** and **sale day**.

| Metric | Healthy interpretation |
|--------|------------------------|
| CPU 60–70% steady | Room for spikes |
| CPU 95% + rising lag | **Fragile** — not “efficient” |
| Disk state growth unbounded | **MTBF** bomb |

---

## Schema change and compatibility — silent CFR multipliers

**Schema registry** rules (BACKWARD, FORWARD, FULL) are **compatibility contracts**.

Breaking change without plan = **CFR event**:

- deserializer failures,
- null fields → wrong defaults,
- downstream SQL **silent** nulls.

| Junior thinking | Senior thinking |
|-----------------|-----------------|
| “Add column — easy.” | “Who consumes? **Compatibility** mode? **Rollout** order?” |
| “Producers deploy first.” | Often need **forward-compatible** readers first |
| “JSON is flexible.” | JSON is **schema drift** with extra steps |

**Real-life analogy:** **USB-C adapter** rollout — one wrong plug fries the **downstream** device.

**Data contracts (teaser):** upstream commits to field presence, semantics, and **SLA to log** — Part 4 topic if you want it.

---

## The senior streaming dashboard

One page. Answers: **“Can we trust live data right now?”**

| Panel | Question |
|-------|----------|
| **p95/p99 lag** per critical consumer | Keeping up? |
| **End-to-end latency** | Stale decisions? |
| **SLO compliance %** (28d) | Budget left? |
| **Checkpoint / commit duration** | Failure brewing? |
| **Backpressure** | Bottleneck where? |
| **Late / dropped events** | Correctness hole? |
| **Duplicate rate / reconciliation delta** | Double-count? |
| **CFR** last 10 deploys | Shipping safely? |
| **State size growth** | Upcoming OOM? |

**Real-life analogy:** Pilot’s **six-pack** — airspeed alone does not land the plane.

---

## Performance reviews for streaming engineers

### Weak vs strong bullets

| Weak | Strong |
|------|--------|
| “Maintained Kafka consumers.” | “Held **fraud stream p99 latency < 500ms** for 99.95% of business hours Q3.” |
| “Upgraded Flink version.” | “Reduced **checkpoint time** 8m → 90s; **MTTR** for lag incidents 2h → 25m.” |
| “Fixed lag alerts.” | “Cut **hot-key skew** — p99 lag down **70%** on sale day; zero SLO breach vs 3 prior quarter.” |
| “Implemented exactly-once.” | “End-to-end **duplicate rate < 0.01%** with idempotent sink + daily reconciliation to batch.” |

### Trade-off story (sounds senior)

> “We increased **allowed lateness** from 5m → 15m for inventory stream — **completeness** up, p99 latency SLO missed **12 minutes total** in March (within budget). Finance signed off after reconciliation spec.”

### Bring to review

- One **SLO chart** (before/after)
- One **incident** with MTTR + MTBF follow-up
- One **deploy** where CFR improved (savepoint discipline, schema rules)
- One **cost / utilization** win without breaching headroom

---

## Quarter plan — streaming edition

| Month | Focus | Target | Freeze if red |
|-------|-------|--------|---------------|
| M1 | Lag SLO on stream A | 99.9% compliance | State schema changes |
| M2 | CFR on job deploys | < 8% bad deploys | Offset reset without runbook |
| M3 | Reconciliation + dup rate | < 0.05% delta | New window types without load test |

**Weekly 15 minutes:**

1. p99 **lag** + **latency** vs SLO  
2. **Error budget** remaining  
3. **Checkpoint** duration trend  
4. Any **partition skew** alert?  
5. One downstream **“does this number look right?”** check  

---

## Conclusion — the senior streaming sentence

**Part 1** taught the scoreboard change. **Part 2** applied it to **batch pipelines**. **Part 3** is the always-on case:

> Junior you: **“The stream processor is running.”**  
> Senior you: **“We meet latency and completeness promises under delay, duplication, and deploy — and we know when we’re spending error budget on purpose.”**

That means:

- **Lag** without **latency** and **correctness** is a vanity metric  
- **Watermarks** and **allowed lateness** are product decisions, not Flink trivia  
- **Exactly-once** is an **end-to-end** story with idempotency and reconciliation  
- **DORA** applies to **savepoint deploys** and **schema evolution**  
- **Reviews** cite **SLO minutes saved**, not “kept Kafka up”  

You do not need a perfect Flink slide deck. You need **one critical stream** with a defined SLO, a budget, a reconciliation story, and a deploy runbook — then you sound senior because you **are** senior.

**Series links:**

- [Part 1 — General metrics mindset](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md)  
- [Part 2 — Batch data pipelines](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/data-pipelines-error-budgets-dora-performance.md)  

**Feedback welcome:** Part 4 could cover **data contracts with upstream producers** or **streaming cost optimization** (rocksdb, changelog, tiered storage) — say which hurts more today.
