<!-- #genai: R2 (coding + system design) interview cram sheet for Docusign Cloud/DevOps role -->

# Docusign R2 — Coding + System Design Cram Sheet

For a **medium-to-senior DevOps/Platform** role they are checking **fundamentals and how you think**, NOT
clean optimal code or a full architecture. It's a conversation. Two power-moves:

1. **Think out loud.** Say your approach before you code. They score reasoning.
2. **Name your gap honestly, then pivot to concept.** *"I haven't built this hands-on, but here's how I'd
   approach it and why."* The recruiter literally said this is what they want.

Read top to bottom. Time budget noted per section.

---

# PART A — CODING FUNDAMENTALS (30 min)

You will likely get **string/array/log-processing** problems (very DevOps-flavored), not hard algorithms.
Language: **Python** (most forgiving to talk through).

## A1 — Data structures: when to use what (5 min)

| Structure | Use when | Lookup |
|---|---|---|
| **List** | ordered items, you iterate / append | O(n) to search |
| **Dict (hash map)** | key → value, fast lookup, counting, grouping | **O(1)** average |
| **Set** | uniqueness, "have I seen this?", dedupe | **O(1)** average |
| **Tuple** | fixed, immutable record | — |

**The #1 instinct to show:** "I'd use a **dict/set** to turn an O(n²) scan into O(n)." Reaching for a hash map
to avoid nested loops is the single most common thing they want to see.

## A2 — Big-O intuition (4 min)

- **O(1)** — constant (dict lookup).
- **O(n)** — one pass over the data.
- **O(n log n)** — sorting.
- **O(n²)** — nested loop over the same data (usually a smell — can a hash map fix it?).

You don't need formal proofs. Just say: *"This is one pass, so O(n) time, O(n) space for the map."*

## A3 — Python idioms to have ready (5 min)

```python
# Count occurrences
from collections import Counter
counts = Counter(items)            # {item: count}

# Group / default values
from collections import defaultdict
groups = defaultdict(list)
groups[key].append(value)

# Read a file line by line (memory-safe for huge logs)
with open("app.log") as f:
    for line in f:                 # streams, doesn't load whole file
        ...

# Useful basics
sorted(data, key=lambda x: x[1], reverse=True)   # sort by a field
", ".join(parts)
"ERROR" in line
line.split()                       # tokenize a log line
```

## A4 — DevOps-style problems (talk through these out loud) (10 min)

**Q1. Count how many times each HTTP status code appears in a log file.**
```python
from collections import Counter

def status_counts(path):
    counts = Counter()
    with open(path) as f:
        for line in f:
            parts = line.split()
            if parts:
                counts[parts[-1]] += 1   # assume status is last field
    return counts
```
Talking track: *"Stream the file so memory stays flat even for GB-size logs — O(n) time, O(k) space where
k is number of distinct codes."*

**Q2. Find all lines containing "404" and print them (your repo already has this one).**
```python
def find_404(path):
    with open(path) as f:
        return [line.rstrip() for line in f if "404" in line]
```

**Q3. First non-repeating character / find duplicates** — show the hash-map instinct:
```python
def first_unique(s):
    counts = Counter(s)
    for ch in s:
        if counts[ch] == 1:
            return ch
    return None
```

**Q4. Two Sum (the classic "use a hash map" question).**
```python
def two_sum(nums, target):
    seen = {}                         # value -> index
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i
    return []
```
Talking track: *"Brute force is O(n²) with nested loops. Using a hash map of complements makes it one pass, O(n)."*

**Q5. Is a string a palindrome / reverse words** — basic two-pointer or slicing (`s[::-1]`).

## A5 — How to handle a coding problem live (6 min checklist)

1. **Restate** the problem + ask 1–2 clarifying questions (input size? edge cases? empty input?).
2. **Brute force first**, out loud. *"The simplest way is..."*
3. **Optimize**: "Can a dict/set remove the nested loop?"
4. **Code it**, narrating.
5. **Test** with one normal + one edge case (empty, single element).
6. **State complexity**: time + space.

If you blank: *"Let me start with the brute-force and improve it"* — always a safe opening.

---

# PART B — SYSTEM DESIGN (conceptual) (30 min)

They are NOT expecting a full design. They want you to **understand an architecture and reason about
trade-offs**. Learn the building blocks + a repeatable framework + 2 worked examples.

## B1 — The building blocks (you compose these in every design) (8 min)

- **Load Balancer (LB)** — spreads traffic across many app servers; enables horizontal scaling + HA.
- **App tier (stateless)** — the servers running your code. **Keep them stateless** (no user data stored
  locally) so any server can handle any request and you can add/remove them freely.
- **Database** — the source of truth.
  - **SQL** (Postgres/MySQL): structured, relations, transactions/ACID.
  - **NoSQL** (DynamoDB/Mongo): flexible schema, scales horizontally, key-value/document.
- **Cache (Redis/Memcached)** — store hot data in memory to take load off the DB and cut latency.
- **CDN** — cache static content close to users (edge).
- **Message Queue (SQS/Kafka)** — decouple services; handle spikes; async/background work.
- **Object storage (S3)** — files, images, backups, logs.

## B2 — Core concepts to say confidently (8 min)

- **Vertical scaling** = bigger machine (simple, has a ceiling). **Horizontal scaling** = more machines
  (needs statelessness + LB; this is how you scale for real).
- **Stateless vs stateful:** push state out (to DB/cache/session store) so the app tier scales freely.
- **High Availability (HA):** no single point of failure — run **multiple instances across Availability
  Zones**, with health checks + failover.
- **Caching trade-off:** fast, but you must handle **staleness/invalidation** (the hard part).
- **CAP theorem (1-liner):** in a network partition you choose **Consistency or Availability**. Most web
  systems lean **AP** (available, eventually consistent).
- **Vertical vs horizontal DB scaling:** **read replicas** for read-heavy load; **sharding/partitioning**
  to scale writes.
- **Async over sync:** use a **queue** for slow/background work so user requests stay fast.
- **Bottleneck instinct:** the **database is usually the bottleneck** → relieve it with caching, replicas,
  and async writes.

## B3 — A framework for ANY design question (4 min)

1. **Clarify scope & requirements** (functional + non-functional: scale, latency, read/write ratio).
2. **Rough scale numbers** (users, requests/sec, data size) — even ballpark shows seniority.
3. **High-level diagram**: Client → LB → App (stateless) → DB, then add Cache / Queue / CDN as needed.
4. **Identify bottlenecks** and how you'd scale each tier.
5. **Trade-offs**: consistency vs availability, cost vs complexity. *"It depends on the requirements"* is a
   correct senior answer.

## B4 — Worked example 1: "Design a scalable web application" (5 min)

> Client → **CDN** (static assets) → **Load Balancer** → **stateless app servers (auto-scaling group,
> multi-AZ)** → **Redis cache** → **database (with read replicas)**. Long jobs go to a **queue** processed by
> **workers**. Files in **S3**. Monitoring via metrics/logs/traces.

Talking track: *"App tier is stateless behind an LB so it scales horizontally across AZs for HA. Reads are
served from Redis to protect the DB; read replicas handle read-heavy load. Background work is offloaded to a
queue so user-facing requests stay fast. The DB is the usual bottleneck, so caching + replicas + async
writes are my first levers."*

## B5 — Worked example 2 (DevOps-flavored): "Design a CI/CD / deployment pipeline" (5 min)

> Developer pushes to **Git** → **CI** (GitHub Actions) runs build + tests, builds a **Docker image**,
> pushes to **ECR**, and updates manifests in a Git repo → **Argo CD** (GitOps) reconciles the change into
> **EKS** → progressive rollout (rolling/canary) → **observability** (metrics/logs/alerts) → rollback by
> reverting the Git commit.

This plays directly to your R1 strengths — if they let you pick a design, **steer toward infra/pipeline or a
logging/monitoring system**, where your DevOps knowledge shines.

**Alt prompt — "Design a log aggregation / monitoring system":**
> Agents on hosts ship logs → **queue (Kafka)** to absorb spikes → processors → **storage (Elasticsearch/S3)**
> → **Grafana/Kibana** dashboards + **alerting**. Trade-offs: retention vs cost, sampling, indexing.

---

# THE 7 THINGS TO MEMORIZE (review last)

1. **Hash map to kill nested loops** → O(n²) becomes O(n). The instinct they want in coding.
2. **Always: brute force → optimize → complexity.** Narrate it.
3. **Stateless app tier + load balancer** = horizontal scaling + HA across AZs.
4. **The DB is usually the bottleneck** → cache, read replicas, sharding, async writes.
5. **Cache trade-off** = speed vs staleness/invalidation.
6. **CAP** = under partition, pick Consistency or Availability; web apps usually AP.
7. **Design framework**: clarify → scale numbers → diagram → bottlenecks → trade-offs.

---

## Honesty scripts (use without shame — the recruiter endorsed this)
- *"I haven't implemented this hands-on, but conceptually here's how I'd approach it…"*
- *"Let me reason from fundamentals…"*
- *"There's a trade-off here — it depends on whether we optimize for X or Y."*

*Good luck — re-read the 7 must-knows right before R2.*
