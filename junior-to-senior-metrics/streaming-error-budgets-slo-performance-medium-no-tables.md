# Your Stream Is “Green” but Finance Is Wrong — Streaming SLOs, Lag, and Senior Thinking (Part 3)

**Part 3 of:** *Stop Optimizing Your Ticket Count* → [Part 1 (general)](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/junior-to-senior-thinking-metrics.md) · [Part 2 (batch pipelines)](https://github.com/chiheb08/medium-content/blob/main/junior-to-senior-metrics/data-pipelines-error-budgets-dora-performance.md)

**Alternative titles for Medium:**
- *Lag Is Not a Number — Streaming Metrics for Engineers Who Want to Sound Senior*
- *Event Time, Watermarks, and Error Budgets: Part 3 for Kafka, Flink, and Spark Streaming Teams*
- *Exactly-Once Is a Story You Tell Stakeholders — Streaming Reliability in Plain Language*

**Subtitle:** Consumer lag, end-to-end latency, watermarks, delivery semantics, and how to present streaming impact — without pretending the tutorial cluster is production.

> **Medium tip:** Paste section headings as **H2** blocks. For tables, use `streaming-error-budgets-slo-performance-medium-no-tables.md` or screenshot from the full article.

---

> **Medium paste version:** Empty slots for tables. Screenshot from full `.md` file.


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


> **📷 Insert table screenshot here (Table 1)**  
> *Core streaming concepts*

<!-- Empty slot -->


### Time and correctness


> **📷 Insert table screenshot here (Table 2)**  
> *Time and correctness*

<!-- Empty slot -->


### Delivery and failure words


> **📷 Insert table screenshot here (Table 3)**  
> *Delivery and failure words*

<!-- Empty slot -->


### Schema and deploy


> **📷 Insert table screenshot here (Table 4)**  
> *Schema and deploy*

<!-- Empty slot -->


### Metrics reminders (Part 1 & 2)


> **📷 Insert table screenshot here (Table 5)**  
> *Metrics reminders (Part 1 & 2)*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 6)**  
> *Batch vs streaming promises — different scoreboards*

<!-- Empty slot -->


**Senior move:** Do not copy batch SLO slides and replace “daily” with “real-time.” Define **which event**, **which path**, **which latency percentile**.

---

## The streaming SLI menu — what to measure

Pick **one primary SLI per consumer-facing stream** — not twelve graphs nobody owns.


> **📷 Insert table screenshot here (Table 7)**  
> *The streaming SLI menu — what to measure*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 8)**  
> *Junior → senior shift*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 9)**  
> *SLO framing*

<!-- Empty slot -->


**Senior question:** “Latency SLO for **whom** — and what happens when we miss: block, degrade, or alert only?”

---

## Event time, processing time, and watermarks

### Two clocks


> **📷 Insert table screenshot here (Table 10)**  
> *Two clocks*

<!-- Empty slot -->


They diverge — always.

### Watermarks (plain language)

A **watermark** is the engine’s bet:

> “We are unlikely to see events much older than **T** now.”

Used to **close windows** and emit aggregates.

**Real-life analogy:** A sports broadcast says **“results are final through minute 80”** — late VAR reviews after that need a **correction policy**, not silent rewrites.

### Junior → senior shift


> **📷 Insert table screenshot here (Table 11)**  
> *Junior → senior shift*

<!-- Empty slot -->


### Allowed lateness (Flink / Structured Streaming)

Explicit rule: “Accept updates up to **10 minutes** late; after that, side output or drop with metric.”

**Senior move:** Metric **dropped / late records** on the dashboard — not only lag.

---

## Completeness and correctness under delay

Streaming systems often choose **approximate now** vs **correct later**.


> **📷 Insert table screenshot here (Table 12)**  
> *Completeness and correctness under delay*

<!-- Empty slot -->


**Completeness SLI (example):** reconciled stream totals within **0.1%** of batch truth daily.

**Real-life analogy:** **Election night projections** vs **official count** — both useful; only one should be in the legal filing.

**Senior sentence for stakeholders:**

> “We optimize for **p99 latency under 1 minute** with **end-of-day reconciliation** to batch; intraday numbers are **directional**.”

That is honesty — not weakness.

---

## Delivery semantics — what you can actually promise

Marketing loves **exactly-once**. Seniors translate:


> **📷 Insert table screenshot here (Table 13)**  
> *Delivery semantics — what you can actually promise*

<!-- Empty slot -->


**Truth:** End-to-end exactly-once is **engineering plus business definition**, not a checkbox on a Flink slide.

**Real-life analogy:** **Bank transfer:**

- At-most-once: money might **vanish** from sender without arriving.
- At-least-once: might **charge twice** without idempotency keys.
- Exactly-once: **one** visible transfer — what users care about.

### What to document

- **Idempotency key** on sink (e.g. `event_id`)
- **Dedup** window length
- **Side effects** outside DB (email, webhook) — often **at-least-once** forever


> **📷 Insert table screenshot here (Table 14)**  
> *What to document*

<!-- Empty slot -->


---

## Error budgets for always-on pipelines

Batch budgets often count **missed days**. Streaming budgets count **bad minutes** or **breach seconds**.

### Example lag budget

- **SLO:** p99 lag ≤ 120s for 99.9% of business hours per month.
- **Budget:** ~0.1% of business minutes ≈ **~26 minutes/month** (order-of-magnitude; calibrate to your calendar).

### What burns streaming budget


> **📷 Insert table screenshot here (Table 15)**  
> *What burns streaming budget*

<!-- Empty slot -->


### Green / yellow / red (streaming)


> **📷 Insert table screenshot here (Table 16)**  
> *Green / yellow / red (streaming)*

<!-- Empty slot -->


**Real-life analogy:** **Air traffic** — minor delays OK within buffer; **runway closure** stops departures.

---

## DORA when deploy means savepoint

Map DORA to streaming releases:


> **📷 Insert table screenshot here (Table 17)**  
> *DORA when deploy means savepoint*

<!-- Empty slot -->


### Safer streaming deploy patterns

- **Savepoint** before stateful change (Flink)
- **Blue/green** consumer on new topic version
- **Canary** partition or shadow sink
- **Compatible** schema changes (Avro/Protobuf rules)


> **📷 Insert table screenshot here (Table 18)**  
> *Safer streaming deploy patterns*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 19)**  
> *Table*

<!-- Empty slot -->


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


> **📷 Insert table screenshot here (Table 20)**  
> *Utilization (Part 1 callback)*

<!-- Empty slot -->


---

## Schema change and compatibility — silent CFR multipliers

**Schema registry** rules (BACKWARD, FORWARD, FULL) are **compatibility contracts**.

Breaking change without plan = **CFR event**:

- deserializer failures,
- null fields → wrong defaults,
- downstream SQL **silent** nulls.


> **📷 Insert table screenshot here (Table 21)**  
> *Schema change and compatibility — silent CFR multipliers*

<!-- Empty slot -->


**Real-life analogy:** **USB-C adapter** rollout — one wrong plug fries the **downstream** device.

**Data contracts (teaser):** upstream commits to field presence, semantics, and **SLA to log** — Part 4 topic if you want it.

---

## The senior streaming dashboard

One page. Answers: **“Can we trust live data right now?”**


> **📷 Insert table screenshot here (Table 22)**  
> *The senior streaming dashboard*

<!-- Empty slot -->


**Real-life analogy:** Pilot’s **six-pack** — airspeed alone does not land the plane.

---

## Performance reviews for streaming engineers

### Weak vs strong bullets


> **📷 Insert table screenshot here (Table 23)**  
> *Weak vs strong bullets*

<!-- Empty slot -->


### Trade-off story (sounds senior)

> “We increased **allowed lateness** from 5m → 15m for inventory stream — **completeness** up, p99 latency SLO missed **12 minutes total** in March (within budget). Finance signed off after reconciliation spec.”

### Bring to review

- One **SLO chart** (before/after)
- One **incident** with MTTR + MTBF follow-up
- One **deploy** where CFR improved (savepoint discipline, schema rules)
- One **cost / utilization** win without breaching headroom

---

## Quarter plan — streaming edition


> **📷 Insert table screenshot here (Table 24)**  
> *Quarter plan — streaming edition*

<!-- Empty slot -->


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
