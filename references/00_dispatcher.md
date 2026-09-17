# Linear Relay Linking — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The Top 1% of Experts Think on Paper—Here's How](https://www.youtube.com/watch?v=VkXMlrvq29o)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Comment-to-Topic Baton Pass

Every sentence in English has a fixed positional division of labor:

* **Topic** — the opening anchor. It is *given* information: the reader already holds it in working memory, so it costs nothing to re-load.
* **Comment** — the closing payload. It is *novel* information, placed in the sentence's stress position (the end), where retention is highest.

Novice procedural writing treats sentences as **independent containers**: each one opens a fresh Topic, drops its Comment, and walks away. The reader is then forced to re-index local state for every line, holding N disconnected islands that were never joined into a derivation.

The **Linear Relay Linking** pattern forbids that. It enforces a rigid baton pass:

> **The Comment (novel payload) of Sentence N becomes the Topic (opening anchor) of Sentence N+1.**

Sentence N+1 therefore *spends* the payload that Sentence N just paid for, promoting it from the stress position back into the anchor position and immediately leveraging it to carry a new payload. The result is an **unbroken chain of causality**: one monotone execution trace instead of a pile of facts.

### Why this dominates for software documentation

Engineering prose is consumed two ways: by a human skimming on a laptop at 2 a.m., and by an LLM agent reading left-to-right with no ability to backtrack cheaply. Both are **state machines over a token stream**. An orphan Topic is a *re-entrant jump*: the reader must flush the derivation stack, re-resolve "what are we even talking about", and rebuild context. That cost recurs per sentence and compounds per page. Relay-linked prose is *monotone*: state carries forward, each sentence inherits the previous sentence's output as its input — the same property that makes a pipeline debuggable.

```text
ANTI-PATTERN — "Orphan Island" prose (no relay)
────────────────────────────────────────────────────────────────
S1  [T: the deploy job]          ............ [C: writes ./build artifacts]
S2  [T: the Dockerfile]          ............ [C: pins node:22-alpine]
         ^^ ORPHAN TOPIC — C1 is dropped on the floor. Reader must re-index.
S3  [T: our health checks]       ............ [C: fail after 30s]
         ^^ ORPHAN TOPIC again. Zero derivation edges between S1–S3.
S4  [T: registry limits]         ............ [C: cap pushes at 1 GB]

Reader model: 4 disconnected islands. No causal spine. Retention ≈ 0.
Dependency graph:     (S1)   (S2)   (S3)   (S4)      ← star / scatter
```

```text
PATTERN — Linear Relay Chain (batons welded)
────────────────────────────────────────────────────────────────
S1  [T1: the deploy job] ─────► [C1: writes ./build artifacts]
                                        │
                            baton pass  │  (C1 promoted to T2)
                                        ▼
S2  [T2: ./build artifacts] ──► [C2: include an unslimmed 1.2 GB image]
                                        │
                                        ▼
S3  [T3: that 1.2 GB image] ──► [C3: exceeds the registry's 1 GB push cap]
                                        │
                                        ▼
S4  [T4: the registry push cap] ► [C4: is why the release pipeline fails]

Reader model: ONE causal spine. Every sentence earns the next one.
Dependency graph:  (S1)─►(S2)─►(S3)─►(S4)      ← linear, walkable, cacheable
```

The invariant, stated as an equation: `Topic(N+1) ≡ Comment(N)`. Break it and momentum dies; honor it and the reader cannot stop reading, because each sentence is the only possible continuation of the last.

---

## 2. Core Transformation Protocols

### Protocol 1 — Baton Pass (End Payload ⇒ Next Opener)
Identify the load-bearing noun phrase in the **stress position** of Sentence N. That phrase, or a definite alias of it, must open Sentence N+1. The baton must be the *same referent*, not a loosely related one — "the cache" ≠ "Redis" unless the link was already established.

### Protocol 2 — Single Payload per Comment
A Comment may launch **exactly one** new referent if it is going to serve as the next Topic. Two novel payloads in one Comment bifurcate the baton and collapse the chain back into a scatter graph — the reader cannot tell which one to carry forward. Split the sentence and chain the two payloads in sequence.

| Bifurcating Comment (two batons) | Split into a clean relay |
|---|---|
| `The migration renames the orders table and also adds a partial index on tenant_id.` | `The migration renames the orders table to order_headers, leaving every foreign key stale. → Those stale foreign keys then require a second migration, which adds a partial index on tenant_id.` |

### Protocol 3 — Orphan Opener Ban
No sentence may open with material that is new to the reader. Every Topic must be one of:
1. the previous Comment, verbatim or as a **definite alias** (`it`, `that image`, `the resulting digest`, `this failure mode`);
2. a term the reader already holds from a prior relay in the same chain;
3. a section-level anchor the reader was told to expect ("The migration described above…").

If a sentence genuinely needs a new opening referent, **splice a relay clause** onto the previous sentence so the new referent enters in the stress position.

| Orphan opener | Relay-linked replacement |
|---|---|
| `Additionally, the retry policy uses exponential jitter.` | `Those connection resets then hit the retry policy, which adds exponential jitter to every backoff.` |
| `There are three things to check before rollout.` | `The rollout gate then checks three things, starting with the schema version.` |
| `Note that S3 bucket policies also apply here.` | `That presigned URL is then subject to the S3 bucket policy, which restricts the caller's source IP.` |

### Protocol 4 — Given-New Ordering Lock
Force the front of the sentence to be *given* and the end to be *new*. Additive connectors (`also`, `additionally`, `furthermore`, `moreover`) are the primary symptom of a broken relay: they signal that the writer gave up on derivation and is stacking facts instead. Replace them with a causal coupler (`therefore`, `which`, `leaving`, `so that`) attached to the carried baton.

| Disjunctive / additive anti-pattern | Relay-linked replacement |
|---|---|
| `The worker also writes a lockfile, and additionally it fsyncs the directory.` | `The worker writes a lockfile, then fsyncs that lockfile's parent directory before exiting.` |
| `There is a race between the two consumers. Also, the broker redelivers unacked messages.` | `That unacked message is then redelivered by the broker, re-entering the same race with the second consumer.` |
| `Validation happens in the schema layer. This means the API can reject early.` | `The schema layer performs validation, which lets the API reject malformed payloads before the handler runs.` |

### Protocol 5 — Referential Precision (Repeat the Baton, Don't Pronominalize It Away)
When more than one candidate referent exists, repeat the exact noun. Pronoun drift (`it`, `this`, `that` with an ambiguous antecedent) is the most common silent chain break in engineering prose, and it is catastrophic for retrieval and for LLM agents that cannot ask a follow-up. Reserve pronouns for chains with a single dominant referent.

| Drifting pronoun | Repaired baton |
|---|---|
| `The shard rebalances, and it then drops the leader lease, so it resigns.` | `The shard rebalances and drops its leader lease, so the primary resigns before the follower drains.` |
| `This causes a panic, which stops it from flushing.` | `That null dereference panics the ingest worker, which stops it from flushing the WAL.` |

### Protocol 6 — Chain-Length Discipline & Checkpoints
A relay chain stays legible for roughly **4–6 links**. Beyond that, insert a **checkpoint sentence** that opens with the chain's original anchor and closes with the current baton, re-grounding the reader without breaking causality: *"The deploy job, by way of that oversized image, is now blocked at the registry."* Checkpoints are mandatory in runbooks and trace walkthroughs longer than ~120 words.

### Protocol 7 — Terminal Closure
The **last** sentence of a step, section, or document must place the *outcome* — exit code, verdict, blocking condition, or the next action — in its Comment. Never let a chain trail off into a trailing side fact; that leaves the reader with an unconsumed payload and no terminal state.

### The Relay Ledger (drafting scaffold)
Draft the chain as a ledger before writing prose. Reading the `Comment` column top-to-bottom must read as the outline; the `Topic` column of row N+1 must equal the `Comment` column of row N.

| # | Topic (given) | Comment (new baton) | Relay ✓ |
|---|---|---|---|
| 1 | the deploy job | writes `./build` artifacts | — |
| 2 | `./build` artifacts | include a 1.2 GB image | ✓ |
| 3 | that 1.2 GB image | trips the registry's 1 GB push cap | ✓ |
| 4 | the registry push cap | blocks the release pipeline | ✓ |

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews (inline walkthrough comments)
A review comment is an *execution trace walkthrough*: it must carry the author along the same path the reviewer took. Ledger-style comments ("Also, this method is O(n²). Additionally, the config isn't validated.") give the author three unrelated tasks and no priority.

**Before (orphan islands):**
> The inner loop calls `findUser` for every row. That's an N+1 query. Also, the response object is missing validation, and we should probably add a timeout on the HTTP client.

**After (linear relay):**
> The inner loop calls `findUser` once per row, turning this into an N+1 query. That N+1 is what pushes p99 to 1.4 s on staging. Those 1.4 s responses then hold the HTTP client open indefinitely, because the client has no timeout — so a single slow query can exhaust the pool. **Requested change:** add a `timeout=` to the client and batch `findUser` into a single `WHERE id IN (...)` lookup.

Every sentence opens with the previous sentence's closing payload; the reviewer's priority is now derived instead of asserted. When the chain would exceed five links, checkpoint: *"The N+1, by way of those unbounded responses, is the root cause of the pool exhaustion."*

### 3.2 PR Descriptions
The summary block should read as a four-link relay: problem → mechanism → effect → risk or rollback.

**Before:**
> This PR updates the billing service. Also adds idempotency keys. There's a new index on `invoices.customer_id`. It should be safe to deploy.

**After (relay chain):**
> Retried webhooks were charging customers twice, so **this PR introduces an idempotency key on every charge request**. That idempotency key is stored in `charge_attempts`, a new table whose primary key is the key itself — so a duplicate insert now fails loudly instead of double-charging. **Those duplicate inserts** surface as a `409` the webhook sender treats as success, which converts the double-charge into a no-op. The new index on `charge_attempts.customer_id` keeps the lookup under 2 ms, and the migration is additive, so **rollback is a single `DROP TABLE`**.

Note that the risk is the *terminal Comment* (Protocol 7): the reader finishes on a known, actionable end state.

### 3.3 Architecture RFCs / ADRs
RFC sections (`Context`, `Decision`, `Consequences`) are chain boundaries, not chain breaks: **the last sentence of Context is the Topic of the first sentence of Decision.** Headers may interrupt typographically, but the relay must survive them.

**Before:**
> **Context.** We currently store sessions in a single Redis instance. Latency spikes during failover. There are also compliance requirements around EU data residency.  
> **Decision.** We will shard sessions by region.

**After:**
> **Context.** A single Redis instance holds every session, so **a failover event stalls all authenticated traffic for 40–90 s**. That stall is amplified by the failover path itself: the promoted replica must load a 12 GB keyset before it accepts writes, extending the stall well past the failover window. **The same keyset** also cannot be region-scoped, which conflicts with our EU data-residency obligation — EU session bytes currently replicate to `us-east-1`.  
> **Decision.** Because that unshardable, cross-region keyset is the root cause, **sessions are sharded by region into three independent Redis clusters**, each with its own EU- or US-resident keyset.  
> **Consequences.** Those three clusters then require per-region connection routing, so the gateway must resolve a session's home cluster from the JWT claim — which adds one lookup per request and pins the routing table to the auth service's release cycle.

The Decision opens with the Context's terminal baton ("that unshardable, cross-region keyset"), and Consequences opens with the Decision's ("those three clusters"). An ADR written this way is re-derivable by an agent reading only the section headers plus the first clause of each section.

---

## 4. Verification Checklist

- [ ] **Baton equality test**: For every adjacent pair of sentences, does Sentence N+1 open with the exact referent (noun or definite alias) that ended Sentence N? If not, found an orphan opener — splice a relay clause.
- [ ] **Additive-connector scan**: Search the draft for `also`, `additionally`, `furthermore`, `moreover`, `there is/are`, and sentence-initial `This/It`. Each hit marks a broken or ambiguous relay rather than a stylistic preference.
- [ ] **Single-payload audit**: Does any Comment introduce two *new* referents? If yes, split it so the chain stays single-track (`Topic(N+1) ≡ Comment(N)`).
- [ ] **Chain-length check**: Is any chain longer than ~6 links without a checkpoint sentence that re-grounds on the original Topic? Add one.
- [ ] **Terminal closure test**: Do the last sentences of every step, section, and document place the outcome (exit code, verdict, blocking condition, next action) in the stress position — leaving no unconsumed trailing fact?
- [ ] **Cold-read derivation**: Read only the Comment column of the Relay Ledger top-to-bottom. Does it read as a coherent causal outline? If it reads as a scatter of unrelated facts, the relay is not welded.