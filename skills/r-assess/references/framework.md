# The R framework — full reference

R (`R0`–`R3`, optional `+`) is a **responsibility rating** set next to Story Points: not how much
work a task holds, but what the person who clicks **Approve** is risking. AI compressed the cost
that SP measured; it did not touch the cost that stays on the human. R makes that remaining cost
visible, plannable, and limitable.

One number lives in three roles, and there is no contradiction: it is **derived** from risk on
three axes, it **prescribes** verification depth, and it is **spent** as responsibility — which is
why it is limited. Input, prescription, and price of the same thing.

## Where R is spent

Two places in the process consume it:

- **DRI** (usually the author). AI did the task; the only question is whether to trust green tests
  or take the solution apart. If you always take everything apart, writing it yourself is cheaper
  and the AI gain disappears. R answers that question.
- **Code review.** The second engineer faces the same fork with less context. AI review is always
  present and cheap, but it lowers the probability of a mistake — not the price of the approve.

### Prescribed depth by level

| R  | DRI                                                                | Code review                                                             |
| -- | ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| R0 | green tests are enough                                             | diff scan, AI review                                                     |
| R1 | read the diff, check the edges                                     | AI review + scan, questions on the edges                                 |
| R2 | take apart the **solution**, not the diff; verify tests test meaning, not just green | read **every line**; AI review is a first pass, not an argument for approve |
| R3 | reproduce manually or dry-run; be able to state the invariant out loud | read in full; approve only after the rollback has been talked through    |

`+` at the same digit means: **treat as one level higher**.

## The three axes

| Axis          | Question                     | High                                   | Medium                              | Low                                    |
| ------------- | ---------------------------- | -------------------------------------- | ----------------------------------- | -------------------------------------- |
| Blast Radius  | who sees it if it's wrong    | all clients, money, public contract    | some clients, one tenant            | only me or the team                    |
| Reversibility | what undoes it               | revert, flag off                       | backfill, reverse migration, manual cleanup | nothing: data deleted, money gone, email delivered |
| Detectability | what fires, and when         | test or alert within an hour           | metrics or support tickets in days  | by accident in weeks, or never         |

Polarity differs: for Blast Radius **High is bad**; for the other two **High is good**.

Axes are judged by the change's **purpose**, not by the environment the code lives in today
(critical for greenfield — see below).

Provenance: Blast Radius is standard SRE / incident-management vocabulary; Reversibility is change
management ("one-way and two-way doors"); Detectability is a first-class risk axis in FMEA
(RPN = Severity × Occurrence × Detection, AIAG-VDA). What is original here: folding three axes
into one plannable number, and putting **limits** on top instead of a sum.

## Notation and the matrix

Eight values: `R0 R1 R2 R3` and the same with `+`. The digit is derived from the pair
**(Blast Radius, Effective Recoverability)** and describes the change itself.

Reversibility and Detectability are not independent — reversibility is useless if you never
noticed you need to roll back. They fold into one value:

> **E = worst(Reversibility, Detectability)**, on the same High/Medium/Low scale.

|                   | E = High | E = Medium | E = Low |
| ----------------- | -------- | ---------- | ------- |
| **Radius Low**    | R0       | R1         | R2 ✳    |
| **Radius Medium** | R1       | R2         | R2      |
| **Radius High**   | R2 ✳     | R2         | R3      |

✳ = the axes contradict each other (wide radius with full recoverability, or irreversibility with
a tiny radius). Disputed cells — **calibrate these first**.

### The `+` rules

- `+` means "worse than the digit says, for a reason **named in the ticket**": treat as one level higher.
- The matrix sets `+` **automatically when Detectability = Low** — you learn too late, and
  downstream controls don't save you. This matrix-`+` can be removed **only by buying detect**.
- A human may add `+` for other named reasons: unfamiliar area, code the DRI doesn't own, first
  change of its kind, external dependency, deadline pressure.
- There is no level above R3, so **R3+ reads differently**: it is the maximum treatment there is,
  and a signal that it's cheaper to **buy reversibility and come back with a smaller number** than
  to strengthen review further.
- **If any axis is unassessed, R is not set. No number — no signature.**

## R is bought down with SP

R describes not the task but **the task in its current design**. The same result can be delivered
by a design with a different price for the human — paid in exactly the resource AI devalued.

**R is set after the purchases, not before**: judge the axes → see what can be bought → buy →
set the final R. Detect can almost always be bought, so buy it **by default** and recompute. A
detect-`+` remains only where buying failed.

| Axis          | You pay with        | Availability                                                                 |
| ------------- | ------------------- | ---------------------------------------------------------------------------- |
| Detectability | SP, usually few     | almost always: alert, invariant assert, reconciliation, metric shipped **before** the behaviour change |
| Reversibility | design              | often, but capped: a delivered email cannot be recalled                      |
| Blast Radius  | calendar time + volume | always, but it's a **product** decision: 1% traffic, one tenant, employees first |

- Detect drops the **level** only when Reversibility ≠ Low (e.g. row 21: R3+ → R2 for the price of
  one alert). If the action is irreversible, detect removes only the `+`; to drop the level you
  must buy reversibility: dry-run, hold queue, cancellation window before sending.
- **Detect purchases accumulate, they are not consumed.** An invariant alert in billing lowers R
  for every future billing task: the first task in the area pays, the rest inherit.
- **A purchase is complete when the alert demonstrably fires**, not when it's promised in the
  ticket description — otherwise review depth is removed without removing risk. Until proof, treat
  the task at the higher (pre-purchase) level.
- On buy-down **SP grows and R falls**, so in velocity the person who reduced risk looks less
  productive. Without explicitly accounting for this trade, nobody will buy.

## Why limits, not a sum

Story Points add up because they approximate time, and time is additive. R approximates depth of
immersion. Two R3 tasks in a week is not the same as three R2: the second deep dive arrives on
spent attention, and DRI responsibility does not divide between people.

Same form as WIP limits in Kanban and pilot duty limits: a **condition**, not a sum.

- Limit **per level**, not per sum. Numbers must be measured; a starting guess is
  **1×R3 and 3×R2 per person per week** — a guess, not a recommendation.
- Limit **concurrency**, not approve dates: two R3 held in the head at once is overload even if
  the approves land on different days.
- **The reviewer spends their limit too.** R3 costs both people. Without this, all R3 silently
  drains to the one strong reviewer.
- **A task without an R is not planned.**
- The week is checked by asking "is anyone over a limit", not by summing.

## System Design as a separate task

R measures spent human attention, not just diff-reading depth. A first System Design ticket
carries R: thinking a solution through spends the same resource, and it can't be handed to AI
turnkey — you pay with rework time and accepted constraints.

The axes are read against a different subject:

| Axis          | On a System Design task                                                            |
| ------------- | ----------------------------------------------------------------------------------- |
| Blast Radius  | who inherits the decision: child tasks, neighbour services, a future migration       |
| Reversibility | cost to revisit later: re-planning + thrown-away work vs a data model already live in prod |
| Detectability | when wrongness surfaces: during implementation (a feature won't fit) or already in prod |

The level prescribes depth of **analysis** and who signs: at R2, constraints are **measured, not
assumed**, and a second competent person reads the whole design (AI analysis is a first pass).

No double-charging: the parent task pays for the decision, children pay for the rollout — distinct
acts of attention in different weeks.

**Epics get no R.** A container spends no attention; its children do. An epic-level number, if
needed for planning, is a roll-up over children for reporting — not a mark.

## Greenfield: judge by purpose

Greenfield = building from scratch: empty repo, no prod data, no contract consumers. The
temptation: while nothing is connected, a mistake harms nobody → everything is R0. **That's the trap.**

A defect in unconnected code is not zero, it is **deferred**: it lies dormant and fires on
connection — classically, a defect accepted long before activation and enabled later by one config
line. And pre-launch reversibility is only high **if someone noticed the defect**: if the code
passed as R0 and nobody read it, neither reversibility nor detect works.

- **Rule: judge by purpose, not by today's environment.** Not "what breaks today" but "whose
  mistake does this become when it goes live". A layer that will hold client data is R2 **on the
  day it is written**. The real distinction is "will we connect it or throw it away": a prototype
  destined to die genuinely is R0.
- **The connection task activates accumulated R but does not absorb it.** Responsibility for the
  code was already spent by those who wrote and accepted it. The switch has its own R — for the
  mechanics only: routing, flag, gradualness, rollback. Otherwise greenfield becomes risk
  laundering: build half a service at R0 and connect it with one R3, having read nothing — and the
  whole service gets read in one sitting, the worst possible mode.
- **Greenfield is a period of high R-load, not low.** Deferred responsibility accumulates faster
  than at any later point, so limits matter most here: one person cannot be DRI for every layer,
  even when each task looks harmless. This is also why the nastiest latent defects live in new services.
- **What greenfield really gives: the cheapest buy-down window.** Before launch there is no data
  and no contract dependency, so changing the design, adding a flag, invariants, and metrics costs
  pennies — and lowers R for every future task in the service at once.

Example — new service, decomposed by layer, not by feature:

| Task                                   | R      | Why                                                                                  |
| -------------------------------------- | ------ | ------------------------------------------------------------------------------------ |
| Skeleton, deploy, health-check         | R1     | fails loudly and is reversible                                                        |
| Abstractions for NFRs and FRs          | R2+    | everything else inherits them; you find out when the third feature doesn't fit; at Low radius this is a ✳ cell — clients don't see it, but it's irreversible and silent |
| Data storage layer                     | R2     | will hold client data; after launch schema changes need a backfill                    |
| Access layer to third-party systems    | R2+    | real outbound calls, limits shared with prod, vendor side effects can't be recalled   |
| API layer                              | R2     | a contract you can't recall later                                                     |
| Tests for the main paths               | R1     | sometimes `+`: bad tests give false confidence, i.e. poison detect for everything else |
| Subscribing to system events, retention| R2+    | touches the live system; consequences are about data deletion                         |
| Traffic enablement                     | R2/R3  | for the switch mechanics; the accumulated part was spent earlier                      |

The expensive tasks are the ones touching the **existing** system, not the new one — and they are
easy to miss next to "build the whole service".

**The `+`-inflation trap:** a new service is by definition "first change of its kind" with no
observability, so Detectability is formally Low everywhere. If you put `+` on everything, it stops
meaning anything. Keep it where you genuinely won't learn about a mistake: integrations,
subscriptions, retention.

## Cheatsheet: all 27 combinations

B = Blast Radius, V = Reversibility, D = Detectability, E = worst(V, D). Base R **before**
purchases; if detect is bought the row changes and `+` goes away.

| #  | B    | V    | D    | E      | R    | ✳ | Example / note                                                       |
| -- | ---- | ---- | ---- | ------ | ---- | -- | --------------------------------------------------------------------- |
| 1  | Low  | High | High | High   | R0   |    | internal admin UI; the only R0 cell of 27                             |
| 2  | Low  | High | Med  | Medium | R1   |    | ·                                                                      |
| 3  | Low  | High | Low  | Low    | R2+  | ✳  | local script whose output nobody cross-checks                          |
| 4  | Low  | Med  | High | Medium | R1   |    | ·                                                                      |
| 5  | Low  | Med  | Med  | Medium | R1   |    | ·                                                                      |
| 6  | Low  | Med  | Low  | Low    | R2+  | ✳  | ·                                                                      |
| 7  | Low  | Low  | High | Low    | R2   | ✳  | irreversible but tiny radius: axes argue                               |
| 8  | Low  | Low  | Med  | Low    | R2   | ✳  | ·                                                                      |
| 9  | Low  | Low  | Low  | Low    | R2+  | ✳  | small, irreversible, silent                                            |
| 10 | Med  | High | High | High   | R1   |    | new endpoint behind a flag                                             |
| 11 | Med  | High | Med  | Medium | R2   |    | ·                                                                      |
| 12 | Med  | High | Low  | Low    | R2+  |    | ·                                                                      |
| 13 | Med  | Med  | High | Medium | R2   |    | ·                                                                      |
| 14 | Med  | Med  | Med  | Medium | R2   |    | ·                                                                      |
| 15 | Med  | Med  | Low  | Low    | R2+  |    | ·                                                                      |
| 16 | Med  | Low  | High | Low    | R2   |    | mailing to one segment, not the whole base                             |
| 17 | Med  | Low  | Med  | Low    | R2   |    | ·                                                                      |
| 18 | Med  | Low  | Low  | Low    | R2+  |    | ·                                                                      |
| 19 | High | High | High | High   | R2   | ✳  | flag rolled out to everyone: all good except the reach                 |
| 20 | High | High | Med  | Medium | R2   |    | ·                                                                      |
| 21 | High | High | Low  | Low    | R3+  |    | config silently drops 2% of events: revert exists, worth nothing       |
| 22 | High | Med  | High | Medium | R2   |    | billing computation                                                    |
| 23 | High | Med  | Med  | Medium | R2   |    | DB migration + backfill                                                |
| 24 | High | Med  | Low  | Low    | R3+  |    | ·                                                                      |
| 25 | High | Low  | High | Low    | R3   |    | mass mailing                                                           |
| 26 | High | Low  | Med  | Low    | R3   |    | ·                                                                      |
| 27 | High | Low  | Low  | Low    | R3+  |    | silent corruption of customer data                                     |

Distribution: R0 = 1, R1 = 4, R2 = 11, R2+ = 6, R3 = 2, R3+ = 3.

Reading the table:

- **Detectability turns everything else into damage.** Row 21: wide radius, easy revert, no detect
  → R3+. You can't roll back what you don't know about. That's more dangerous than irreversible
  but loud (row 25), where damage is at least bounded by your reaction speed.
- **R0 exists in one cell of 27.** If half the backlog looks like R0, the radius is understated.
- **The middle is fat, and should be.** R2/R2+ covers 17 cells: "read every line" is the normal
  mode for anything clients see.
- **Six disputed ✳ cells** (3, 6, 7, 8, 9, 19), all where axes contradict. Calibrate those; the
  rest are boring.

## Measurement (context for the calibrate mode)

- Data must be **automatic and behavioural**, never surveys ("did you read carefully at R3" has a
  known right answer — it measures loyalty, not depth).
- **R is not validated by incidents.** A working control destroys the correlation you'd validate
  it with: high R gets attention precisely because mistakes are expensive, so it doesn't fail more
  often. What is checked instead: was the scale applied where prescribed, and did limits hold.
- Useful flow signals: time in In Review (median + p90 **per R**), p90/p50 spread within one R,
  reject / changes-requested events on PRs. Diff size is collected as a **control variable**, not
  a signal — otherwise every metric correlates with size.
- Lead visibility: share of R2/R3 approves landing on the top-1 reviewer (concentration); how many
  people can review at R3 per area (bus factor); R3 wait time when those people are busy.
- **Never**: per-person productivity metrics, KPI on review time (instantly optimizes into a
  rubber stamp), R in performance review, R as task importance, a lead who sets R alone.
- For any of this to work, R must be **queryable** in the tracker — hence the Jira label
  (`r0`…`r3-plus`) set at creation time.

## Open questions (unresolved by design)

- Does R add up in any form, or are limits the only working shape? No data.
- Are limits equal across people? Probably not — a function of experience in the specific system,
  not of grade.
- Does `+` dilute if reasons multiply? The named-reason-in-ticket requirement is the only guard.
- Is the DRI limit equal to the reviewer limit? The reviewer spends less per task but collects the
  whole team's flow.
- How to observe "the control held" rather than "fewer incidents"? Flow behaviour is the best
  available, and it's thin.

The construction is original — there is no external framework with this name. Limit numbers are
placeholders until calibrated on own data. The buy-R-for-SP idea belongs to Viktor.
