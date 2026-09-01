# Ticket Decomposition

**A method for breaking work into a story plus small, individually-estimated tickets — using
explicit uncertainty classification and timeboxed spikes instead of padded guesses.**

The goal is not accurate estimates. Those are impossible. The goal is **estimates whose error
becomes visible early**. A plan where a wrong assumption surfaces at hour 2 is enormously more
valuable than one where it surfaces at hour 24, even if both were equally wrong at the outset.

This document is self-contained: the method, the calibration procedure, worked examples across four
domains, and a script for deriving a baseline from your own tracker history.

---

## Contents

1. [The one idea that matters](#1-the-one-idea-that-matters)
2. [Procedure](#2-procedure)
   - [2.1 Establish the calibration basis](#21-establish-the-calibration-basis)
   - [2.2 Write the story](#22-write-the-story)
   - [2.3 Draft the ticket list](#23-draft-the-ticket-list)
   - [2.4 Classify uncertainty](#24-classify-uncertainty)
   - [2.5 Estimate, multiply, round](#25-estimate-multiply-round)
   - [2.6 Apply the uncertainty cap](#26-apply-the-uncertainty-cap--the-core-safeguard)
   - [2.7 Write the ticket bodies](#27-write-the-ticket-bodies)
   - [2.8 Budget the sprint above the ticket sum](#28-budget-the-sprint-above-the-ticket-sum)
   - [2.9 Check the plan](#29-check-the-plan-before-presenting-it)
3. [Domain notes](#3-domain-notes)
4. [Output format](#4-output-format)
5. [Calibration — deriving a baseline from history](#5-calibration--deriving-a-baseline-from-history)
6. [Worked examples](#6-worked-examples)
7. [Appendix — calibrate.py](#7-appendix--calibratepy)

---

## 1. The one idea that matters

**Novelty predicts overrun, not size.** A 6h ticket doing something the team has done twenty times
lands near 6h. A 4h ticket doing something nobody has attempted lands anywhere from 4h to 40h.

Most estimation systems pad by *size* — a flat multiplier on everything — which makes routine work
pessimistic and novel work still catastrophically optimistic. This method pads by **uncertainty**
instead, and refuses to attach a large number to work whose scope is genuinely unknown. That refusal
is the whole mechanism; everything else is scaffolding around it.

---

## 2. Procedure

### 2.1 Establish the calibration basis

Before estimating anything, find out what an hour means *for this team*.

- If there is issue-tracker history (Jira, Linear, GitHub Issues), use it. Query resolved issues
  carrying **both** an original estimate and logged time — that pair set is the only thing worth
  calibrating on. See [§5](#5-calibration--deriving-a-baseline-from-history) and the script in
  [§7](#7-appendix--calibratepy).
- If there is a written estimation baseline already, follow it and say so. Its scale and rules take
  precedence over the defaults here.
- If there is nothing, use the defaults below and **say plainly that they are uncalibrated starting
  points**, not measurements.

Do not stall on this. If calibration data isn't readily available, note the gap and proceed with
defaults — an uncalibrated plan delivered now beats a calibrated one delivered next week.

### 2.2 Write the story

One story = one coherent theme with a single outcome. It carries the context that individual tickets
shouldn't repeat:

- **Why now** — the problem, what prompted it, what changes when it's done.
- **Blocking prerequisites** — things that must exist first but are owned elsewhere or unestimated.
  Name these explicitly. Unnamed prerequisites are the most common source of silent sprint overrun,
  because everyone assumes someone else is handling them.
- **Approach decisions already made** — so tickets don't relitigate them.

If the work has no single theme, it is more than one story. Say so rather than forcing it.

### 2.3 Draft the ticket list

Decompose until each ticket is **independently verifiable** — someone can look at it and say done or
not done without running the whole system.

Split **vertically** (a thin slice checkable end to end) rather than **horizontally** (one ticket per
layer). Three tickets for UI, service, and schema that can only be verified together are one ticket
wearing a disguise, and they hide integration risk until all three are "done".

Sequence in dependency order and say which tickets are parallelisable.

### 2.4 Classify uncertainty

Assign every ticket one of three classes. Be honest — this is where the value is, and optimism here
is what the whole method exists to counteract.

| Class | Means | Multiplier |
|---|---|---|
| **Low** | Known pattern, code you have touched, you can name the files before starting | 1.15× |
| **Medium** | New component in a familiar subsystem; a bug with a reliable repro; a library you've used before applied in a new way | 1.5× |
| **High** | Unknown approach, novel integration, unfamiliar toolchain or platform, first use of an API, anything phrased as "investigate", "research", or "figure out" | 2.0× |

Signals that force **High** regardless of how small it feels: nobody on the team has done it before;
it depends on a system whose source you cannot read; it involves new hardware, a new build target, or
a new vendor SDK; the acceptance criteria can't be written yet.

### 2.5 Estimate, multiply, round

1. Estimate the **known** version of the task at face value, using the anchors below.
2. Multiply by the uncertainty factor.
3. Round **up** to the allowed scale: `0.5, 1, 2, 3, 4, 6` hours.
4. If the result exceeds 6h, **split the ticket** — never round down to fit.

Nothing goes below 0.5h; smaller items fold into a parent. Nothing goes above 6h; beyond that,
estimates stop being estimates and become wishes.

**Face-value anchors** (before multipliers):

| | |
|---|---|
| **0.5h** | A trivial bounded change. Config value, one-line fix, a rename. No new surface. |
| **1h** | One bounded change to code that already exists. Single function, field, endpoint, or behaviour. You know which file before you start. |
| **2h** | One new bounded unit, fully specified. A new module, endpoint, UI section, or test scenario. New surface, but no open questions. |
| **3–4h** | A slice spanning layers, or wiring together components that exist but have never been connected. |
| **6h** | A full vertical slice with a known approach. The ceiling — reach for a split first. |

Most tickets should land at 1h or 2h. If most of yours are 4h and 6h, the decomposition is too coarse
and the plan is hiding risk.

### 2.6 Apply the uncertainty cap — the core safeguard

> **A Medium- or High-uncertainty ticket may not carry an estimate above 2h.**

If the face-value work is bigger than that, there are two options and only two:

- **Split it** into ≤2h units each verifiable independently, or
- **Write a spike** — a timeboxed investigation whose *deliverable is the estimate* for the remaining
  work.

The reasoning: a large number on unknown work doesn't communicate "this is big", it communicates "we
have this under control", and the error stays invisible until the overrun has already accumulated.
The classic failure is a 5h ticket for genuinely novel work that lands at 24h — and no multiplier
survives that. The fix was never a bigger multiplier; it was refusing to let novel work carry a
confident number at all.

**Spike anatomy:**

```
### SPIKE-n · [AREA] Spike: <question being answered> · 2h · High

**Timeboxed. Stop at 2h regardless of state.**

Context: <why the scope is currently unknown>

Scope:
- <the cheapest experiment that produces information>
- Record every distinct failure / finding.

Explicitly NOT in scope: making it work, polish, any downstream ticket.

Done when:
- A written findings list exists in the ticket.
- Estimates for T<x>–T<y> are revised on the back of it — this is the
  deliverable, not the code.
- Throwaway artifacts are discarded, not merged.
```

Mark every downstream ticket the spike feeds as **provisional** until it completes. Show provisional
estimates anyway so the sprint can be shaped — just label them honestly.

### 2.7 Write the ticket bodies

Match the house title convention if one exists (look at existing tickets). A common and good default
is a component prefix plus a terse noun phrase:

```
[BUILD] Add release configuration to CMake preset
[FW]    Guard USB init behind config symbol
[TEST]  Scenario — switch held blocks lock
```

Body template:

```
### T<n> · `[AREA] Title` · <estimate>h · <uncertainty>

<One or two lines of context: why this exists, what it unblocks.>

Scope:
- <concrete change>
- <concrete change>

Done when:
- <checkable condition>
- <checkable condition>
```

**Acceptance criteria must be checkable, not judged.** "Works correctly" and "looks good" are not
acceptance criteria — they defer the argument to review time. Prefer conditions someone can
mechanically confirm: a specific log line appears, a diff is empty, a named test passes, a value
falls in a range, an artifact exists.

Where a mistake would be silent, say *how* to check rather than *what* to check. "Diff the generated
output against the reference" beats "make sure it matches" when eyeballing is exactly what fails.

Include a **failure rule** on tickets where overrun is the signal that matters: *"if this exceeds its
estimate, re-spike rather than continue."* Without that, the cap protects nothing — people just
grind.

### 2.8 Budget the sprint above the ticket sum

Per-ticket padding does not absorb aggregate drift, because the tickets that blow up are not the ones
you padded. For a plan that is mid-to-high uncertainty overall, budget **1.3–1.5× the summed
estimates** on top of the padding already applied.

Present this so the sum and the budget are both visible:

| Block | Tickets | Sum |
|---|---|---|
| Spike | SPIKE-1 | 2h |
| *(block)* | T1–T5 | 9h |
| **Total** | | **11h** |

Then state the budgeted figure and why. If you used a historical overrun ratio, say whether it is a
floor or a central estimate — see [§5.3](#53-the-backfill-trap), since most tracker history
understates overrun for structural reasons.

Call out the **milestone worth landing alone** — usually the block that retires the most risk.

### 2.9 Check the plan before presenting it

- Does any Medium/High ticket exceed 2h? Split it or spike it.
- Is there an "absorber" ticket — integration, bring-up, polish, testing — that exists to soak
  unknowns? Replace it with a spike.
- Can each ticket be verified without its siblings?
- Are prerequisites named, including ones owned by other people?
- Is the happy path the only path estimated? Error handling, migration, rollback, and docs are work.
- If nearly every ticket is Low uncertainty, is that true, or is it optimism?

---

## 3. Domain notes

**Firmware / embedded.** Hardware availability is a scheduling dependency, not an estimate — a shared
bench, rig, or board goes in prerequisites, never inside a ticket's hours. Board bring-up, new
silicon, a new build target, or a vendor SDK upgrade is **always High**, no matter how routine the
code looks, because the failure modes live in toolchain and configuration rather than logic. The
flash-and-debug loop inflates iteration count, so work needing many cycles costs more per unit of
change than the same work on a host. Hardware-in-the-loop tests are slow and flaky enough that "add a
test" is not a 1h ticket. Where the work is a port to a new target, the first build attempt is the
canonical spike.

**C++ / desktop.** Build-system and dependency work (CMake, vcpkg, Conan, packaging, code signing) is
high-novelty far more often than it feels and is the most reliably underestimated category.
Cross-platform means multiplying, not sharing — do not assume one platform's ticket covers the
others; give each its own, because the failures are platform-specific. Separate UI-framework wiring
from business logic; they have different risk profiles and different verifiability. Linker and ABI
problems are tail risk: rare, and enormous when they land.

**Learning / interview prep.** Tickets are study units, and the cap matters more here than anywhere
because attention degrades — keep them at 1–2h. "Done when" should be a **demonstrable artifact**,
not time served: implement X from scratch without references in N minutes; explain the trade-off
between A and B unprompted; solve K unseen problems of type Y unaided; write a one-page summary from
memory. A topic never touched gets a survey spike first, because you cannot decompose what you cannot
yet see the shape of.

---

## 4. Output format

Deliver a single document:

1. **Story** — title, context/why, blocking prerequisites, approach decisions.
2. **Estimation method** — the scale, the multipliers, the calibration basis (or that there is none),
   and which tickets are provisional pending a spike.
3. **Tickets** — in dependency order, each with title, estimate, uncertainty, context, scope,
   done-when.
4. **Sprint shaping** — the sum table, the budgeted figure, the standalone milestone.
5. **Out of scope** — anything found along the way that is real but not this story. Naming it stops
   it being silently absorbed.

**Creating the tickets in a tracker.** Present the breakdown for review first and wait for a clear
yes before creating issues. Creating them is visible to a whole team and tedious to unpick, so it is
not a step to take on inferred approval. Ask which project, issue type, and parent they should land
under rather than guessing.

---

## 5. Calibration — deriving a baseline from history

Naive analysis of estimate-vs-actual data is not merely imprecise, it is usually **misleading in a
specific and predictable direction**. The main job of this section is to stop you reporting a number
that is confidently wrong.

### 5.1 What to pull

Query resolved/closed issues from the last 3–12 months. Longer windows drift as the team and codebase
change; shorter ones give too few paired samples.

| Field | Why |
|---|---|
| Key / ID | Identifying outliers so a human can go read them |
| Summary / title | Inferring work class, and learning the title convention |
| Original estimate | Half the pair |
| Time spent / logged | The other half |
| Issue type | Bug vs task vs spike behave differently |
| Labels / components | Segmenting by subsystem |
| Resolved date | Checking for drift over time |
| Assignee | **Only** for spotting single-person artefacts — never for individual comparison |

The only rows that calibrate anything are those with **both** an estimate and logged time. Expect
this paired set to be much smaller than the total — often a quarter or less. Report the paired count
prominently, because it is the real sample size; a baseline built on 20 pairs deserves far less
confidence than one built on 200.

### 5.2 What to compute

**Aggregate ratio** — the headline number:

```
aggregate_ratio = sum(actual) / sum(estimate)
```

Use the sum of both columns, not the mean of per-ticket ratios. Summing weights each ticket by its
size, which is what you want when budgeting a sprint. Averaging per-ticket ratios lets a 0.5h ticket
that ran 1h (2.0×) distort the picture as much as a 6h ticket that ran 6h.

Also compute: paired sample count; the distribution of estimate values actually used; the share
within ±10% of estimate; per-size-bucket ratios; per-work-class ratios; and the worst five overruns
by ratio and by absolute hours.

### 5.3 The backfill trap

**This is the most important part of the whole document.**

In most teams, logged time is not measured — it is reconciled to the estimate at the end, or entered
in one lump when the ticket is closed. The signature is unmistakable:

- The **median per-ticket ratio is exactly 1.00**, often in every size bucket simultaneously.
- A large share of tickets — frequently more than half — land within ±10% of estimate.
- The largest tickets are the most suspiciously exact: 8h→8h, 16h→16h.

Real durations do not cluster that tightly on a predicted value. Human work is roughly lognormal;
even a well-estimated task lands within ±10% maybe a third of the time. Tight clustering on 1.00 is
evidence of a *process*, not of accuracy.

Three consequences, all of which should be stated when reporting:

1. **Never quote the median.** It measures the backfill convention, not the work.
2. **The aggregate ratio is a floor, not a central estimate.** Backfill pulls values toward 1.00, so
   true overrun is higher than measured by an unknown margin. A measured 1.13× means "at least
   1.13×", and should be presented that way.
3. **The tail is the trustworthy part.** Nobody backfills a 4.8× overrun by accident. When someone
   logs 24h against a 5h estimate, that number is real — it survived the pressure to tidy it up.
   Weight the outliers heavily and the centre lightly.

Compute the ±10% share explicitly and report it. Above roughly 40%, say clearly that the history is
backfilled and every derived figure is optimistic.

**Improving it is cheap:** populate original estimates on every ticket, and log actuals from a real
record rather than reconciling afterwards. Even rough half-hour honesty beats exact-match backfill.
Two sprints of that produce a variance history worth far more than any further analysis of the
existing data.

### 5.4 Segmenting by work class

Size buckets usually show a **flat** ratio — size alone does not predict overrun. That is a genuinely
useful negative result: it means padding by size is wasted effort.

Segment by novelty instead:

| Class | Typical markers |
|---|---|
| **Routine / cosmetic** | copy changes, static UI, config, styling, small bounded fixes |
| **Known-pattern feature** | a new endpoint, a new module of a familiar kind, standard CRUD |
| **Novel / integration** | migrations, infrastructure, build systems, new hardware or platform, third-party or vendor SDK integration, ML/AI work, anything titled "investigate", "research", "spike", or "explore" |

The novel bucket typically runs meaningfully hotter. When it does, you have empirical support for
uncertainty-based rather than size-based padding — which is the premise of this whole approach, so it
is worth checking rather than assuming. If the segments come out equal, say so and fall back to the
default multipliers. Do not manufacture a finding the data does not support.

### 5.5 Reading the tail

Go and read the worst three to five overruns. For each, ask:

- Was it **novel work carrying a confident estimate**? The dominant pattern, and exactly what the
  uncertainty cap prevents.
- Was it **oversized at creation**? Tickets above the cap tend to be both the worst overruns and the
  least trustworthy data points, since large tickets attract the tidiest backfill.
- Was it **blocked on something external** — a person, a shared rig, an approval? That is a
  prerequisite that went unnamed, not an estimation error, and no multiplier fixes it.

The distinction matters because the three have different fixes: cap and spike, split, or name the
prerequisite. Reporting "we underestimate by 30%" when the real cause is unnamed dependencies leads
the team to pad uniformly and change nothing.

### 5.6 Deriving the scale and anchors

**Scale.** Use the estimate values the team actually uses, not an idealised set. If they only ever use
1, 2, 4, and 8, that is the scale; imposing Fibonacci on them will be ignored. Adopt their scale and
apply the 6h cap on top.

**Anchors.** Pull three to five real ticket titles from each size bucket and use those as the
definition of what that size means. Concrete local examples beat any abstract description, because
"1h" is a statement about *this team's* codebase and tooling.

**Cap.** If the history has few tickets above 6h, the team already works this way and the cap is
descriptive rather than imposed — say so, it makes the rule much easier to adopt. Check whether the
compliant subset has a better ratio than the non-compliant one; it usually does, which is the
argument for the cap.

### 5.7 When there is no history

Say so explicitly rather than quietly using defaults as though they were measured. Then use:

- Scale `0.5, 1, 2, 3, 4, 6`, max 6h, min 0.5h.
- Multipliers 1.15× Low, 1.5× Medium, 2.0× High.
- Sprint budget 1.3–1.5× the ticket sum for mid-to-high uncertainty plans.

These are conservative starting points chosen to fail safe, not measurements. The first sprint run
against them is itself the beginning of a baseline.

### 5.8 Reporting the baseline

Include, in this order: sample (window, total, paired count); **data quality** (±10% share and an
explicit backfill verdict — put this *before* the ratio so the caveat is read first); aggregate ratio
labelled floor or central; per-class ratios if they differ; the tail with tickets named; the derived
rules; and what would improve the baseline.

Keep caveats attached to the numbers rather than in a footnote. A ratio quoted without its
data-quality context will be treated as fact and propagated into plans that cannot support it.

---

## 6. Worked examples

Four decompositions, each contrasting a plausible-but-flawed breakdown with a corrected one. The
flaws are the instructive part — every "before" here is the kind of plan that gets approved without
objection and then overruns.

### 6.1 Firmware — porting to a new build target

**Request:** *"We want the firmware to build for a host-simulated target so we can test without the
hardware rig. How long?"*

**Before:**

```
T1  Create board overlay / config          8h
T2  Get it building                        6h
T3  Build test harness                     8h
T4  Write tests                            6h
                                    Total 28h
```

Every ticket is over the cap, and T2 is the clearest tell: **"get it building" is not a task, it is a
hope.** Nobody has attempted a build for this target, so its scope is unknown by definition — yet it
carries a confident 6h. That number will not move until the day it becomes 20h.

**After:**

```
SPIKE-1  [BUILD] Spike: does it configure and link for the new target?   2h  High
T1       [CFG]   Board overlay — inputs                                  2h  Medium
T2       [CFG]   Board overlay — outputs                                 1h  Low
T3       [CFG]   Board overlay — analog channels                         2h  Medium
T4       [CFG]   Config fragment — disable platform-only features        2h  Medium
T5       [FW]    Guard platform-specific init calls                    0.5h  Low
T6       [BUILD] Target boots to clean init                              2h  Medium
T7       [FW]    Harness — input injection commands                      2h  Medium
T8       [FW]    Harness — output readback                               1h  Low
T9       [TEST]  Test scaffold + runner integration                      2h  Medium
T10      [TEST]  Scenario — <specific behaviour>                         1h  Low
                                                                 Total 17.5h
                                                     Budgeted (1.4×)  ~25h
```

The unknown got a **2h spike whose deliverable is the estimates for T1–T6**, not working code. If the
port is impossible or needs a week, that is known at hour 2. T2 (formerly "get it building") became
T6 at 2h *because it now runs after the spike* — the breakage list is known rather than guessed — and
carries a failure rule: exceed 2h, re-spike. The 8h overlay ticket split along a natural seam, each
part verifiable by checking generated output.

The budgeted total lands roughly level with the original 28h, which is the honest outcome. **The
method is not for making plans look smaller; it is for making them survive contact.**

If the host toolchain doesn't exist yet, that is its own item, owned elsewhere, unestimated here.
Burying it inside T1 is how a sprint quietly loses two days. Likewise the bench/rig is a shared
resource and appears in prerequisites, never inside hours.

### 6.2 C++ desktop — a cross-platform feature

**Request:** *"Add file drag-and-drop to the editor. Should work on Windows, macOS and Linux."*

**Before:**

```
T1  Implement drag and drop        6h
T2  Test on all platforms          2h
```

"Test on all platforms" treats three platforms as one unit of work, when drag-and-drop is precisely
the kind of feature whose platform integrations differ completely. And T1 assumes one implementation
covers all three — the assumption most likely to be wrong.

**After:**

```
SPIKE-1  [UI]    Spike: framework drag-drop support across our 3 targets  2h  High
T1       [UI]    Drop-target widget + internal event plumbing             2h  Medium
T2       [UI]    Platform integration — Windows                           2h  Medium
T3       [UI]    Platform integration — macOS                             2h  Medium
T4       [UI]    Platform integration — Linux                             2h  Medium
T5       [CORE]  File-type validation and rejection handling              1h  Low
T6       [BUILD] Packaging — entitlements / manifest changes              2h  High
T7       [TEST]  Manual test matrix, 3 platforms                          1h  Low
                                                                   Total 14h
```

**One ticket per platform** — not padding; the failures are genuinely different per platform, and a
single ticket cannot be partially done in a way anyone can verify. The spike answers the one question
that determines everything downstream: does the framework abstract this, or is it three native
implementations? That answer changes T2–T4 from 2h to 4h+ each, so it is worth 2h to learn first.

**T6 exists at all** because drag-and-drop touches OS-level permissions and app manifests. Packaging,
signing and entitlement work is the most reliably forgotten category in desktop C++, and it is High
because failures surface only in a signed build on a clean machine. T5 is the unhappy path the
original plan folded invisibly into T1.

### 6.3 Bug fix — an intermittent failure

**Request:** *"Users occasionally see corrupted output. Happens maybe once a day. Fix it."*

**Before:**

```
T1  Fix output corruption bug      4h
```

You cannot estimate a fix for a cause you have not found. The 4h is a guess about the *fix*, while
all the risk lives in the *diagnosis* — which could take ten minutes or three days.

**After:**

```
SPIKE-1  [CORE]  Spike: reproduce output corruption reliably       2h  High
T1       [CORE]  Fix — <root cause, filled in by the spike>        ?   provisional
T2       [TEST]  Regression test reproducing the original failure  1h  Low
```

The work splits at the real boundary: **finding it** versus **fixing it**. Only the first can be
estimated up front. If the spike fails to reproduce in 2h, that is itself the finding, and the next
step is a separate ticket for instrumentation rather than more staring.

T1 is deliberately left unestimated. **Writing `?` is more honest and more useful than writing `4h`**
— a question mark prompts a conversation; a wrong number ends one. T2's "done when" is that the test
fails against the pre-fix commit, otherwise it proves nothing.

Bugs with a **reliable repro** are different: Medium at worst, often 1h. The uncertainty in bug work
lives almost entirely in reproduction.

### 6.4 Interview prep — an unfamiliar topic

**Request:** *"I need to get good at concurrency for interviews. Make me a plan."*

**Before:**

```
T1  Study concurrency        3 days
```

Not decomposable, not verifiable, and "3 days" measures time served rather than capability gained —
the thing you actually care about.

**After:**

```
SPIKE-1  [PREP]  Survey the topic and write the real tickets           2h  High
T1       [PREP]  Memory model + happens-before                         2h  Medium
T2       [PREP]  Mutexes, condition variables — implement from scratch  2h  Medium
T3       [PREP]  Lock-free basics: atomics, CAS, ABA                    2h  High
T4       [PREP]  Common interview problems — producer/consumer set      2h  Medium
T5       [PREP]  Deadlock: detection, avoidance, discussion             1h  Low
T6       [PREP]  Mock interview — explain trade-offs unprompted         1h  Medium
                                                                 Total 12h
```

**The survey spike comes first** because you cannot decompose a topic whose shape you cannot yet see;
the tickets above are the spike's *output*, shown here only to illustrate the shape. Every ticket
caps at 2h — attention degrades, and a 4h study block is two mediocre hours wearing a costume.

**"Done when" is a demonstrable artifact, never time served:** implement a condition variable from
scratch, no references, under 30 minutes (T2); explain the ABA problem and one mitigation out loud
without notes (T3); solve three unseen problems unaided (T4); a listener who knows the topic cannot
find a gap (T6).

T3 is High even at the same 2h: lock-free work is where confident study plans quietly fail. Here
uncertainty predicts *needing more sessions*, not a longer session.

### 6.5 Patterns visible across all four

1. **The first ticket is a spike whenever the work is genuinely new.** In all four, the single
   largest risk was "we don't yet know what this involves", and in all four the spike costs 2h to
   retire it.
2. **The oversized ticket is always hiding a seam.** "Get it building", "implement drag and drop",
   "fix the bug", "study concurrency" — each split naturally once the seam was found. A ticket that
   resists splitting usually has unknown scope, which means it wants a spike, not a bigger number.
3. **The forgotten ticket is usually the unhappy path or the packaging.** Validation, error handling,
   entitlements, regression tests, migration. Real work, and their absence is the quiet half of most
   overruns.
4. **`?` is a legitimate estimate for work behind a spike.** It communicates the actual state of
   knowledge. A fabricated number communicates false confidence, which is precisely what you lack.
5. **Honest plans are often not smaller.** The corrected versions here mostly cost the same or more.
   The gain is not a lower number — it is that the number stops being fiction, and that when it is
   wrong you find out in hours rather than weeks.

---

## 7. Appendix — `calibrate.py`

Derives a baseline from a CSV export of resolved issues. Standard library only; auto-detects common
Jira / Linear / GitHub column names.

```bash
python calibrate.py issues.csv
python calibrate.py issues.csv --units seconds     # Jira API exports
python calibrate.py issues.csv --estimate-col "Original Estimate" --actual-col "Time Spent"
```

Its most useful output is the backfill check described in [§5.3](#53-the-backfill-trap) — it says
explicitly when the history cannot support the number you were about to quote.

```python
#!/usr/bin/env python3
"""
Derive an estimation baseline from a CSV export of resolved issues.

Reads estimate/actual pairs, reports the aggregate overrun ratio, and — most
importantly — checks whether the logged time looks *measured* or *backfilled*.
Backfilled worklogs make every derived number optimistic, so that check gates
how much confidence the rest of the output deserves.
"""

import argparse
import csv
import re
import sys
from collections import defaultdict

# Column names seen in Jira / Linear / GitHub exports. Matched case-insensitively
# against a normalised form, so "Σ Original Estimate" and "original_estimate" both hit.
ESTIMATE_HINTS = [
    "originalestimate", "timeoriginalestimate", "estimate", "estimatedhours",
    "estimated", "sumoforiginalestimate",
]
ACTUAL_HINTS = [
    "timespent", "timetracking", "loggedtime", "actualhours", "actual",
    "sumoftimespent", "spent",
]
KEY_HINTS = ["key", "issuekey", "id", "identifier"]
TITLE_HINTS = ["summary", "title", "name"]
TYPE_HINTS = ["issuetype", "type"]
LABEL_HINTS = ["labels", "label", "components", "component"]

# Words that mark work as novel. Novelty predicts overrun far better than size,
# so this segmentation is usually the most actionable part of the report.
NOVEL_MARKERS = [
    "spike", "investigate", "research", "explore", "poc", "prototype",
    "migrat", "integrat", "infra", "bootstrap", "bring-up", "bringup",
    "upgrade", "port ", "porting", "new platform", "toolchain", "pipeline",
    "ml", "ai ", "llm", "experiment",
]
ROUTINE_MARKERS = [
    "copy", "text", "label", "styling", "css", "cosmetic", "rename",
    "typo", "tooltip", "icon", "color", "colour", "padding", "spacing",
]

BUCKETS = [
    ("<=0.5h", 0.0, 0.5),
    ("1h", 0.51, 1.0),
    ("2h", 1.01, 2.0),
    ("3-4h", 2.01, 4.0),
    ("5-6h", 4.01, 6.0),
    (">6h", 6.01, float("inf")),
]


def norm(s):
    return re.sub(r"[^a-z0-9]", "", (s or "").lower())


def find_col(fieldnames, hints, explicit=None):
    """Resolve a column by explicit name first, then by hint matching."""
    if explicit:
        for f in fieldnames:
            if norm(f) == norm(explicit):
                return f
        sys.exit(f"error: column {explicit!r} not found. Available: {', '.join(fieldnames)}")
    normalised = {f: norm(f) for f in fieldnames}
    for hint in hints:  # exact match wins over substring
        for f, n in normalised.items():
            if n == hint:
                return f
    for hint in hints:
        for f, n in normalised.items():
            if hint in n:
                return f
    return None


def parse_duration(raw, units):
    """Parse a duration cell into hours. Handles bare numbers and Jira's '2d 4h 30m'."""
    if raw is None:
        return None
    s = str(raw).strip()
    if not s or s.lower() in {"none", "null", "-", "n/a"}:
        return None
    try:
        val = float(s)
    except ValueError:
        # Compound form: 1w 2d 3h 30m
        mult = {"w": 40.0, "d": 8.0, "h": 1.0, "m": 1.0 / 60.0, "s": 1.0 / 3600.0}
        parts = re.findall(r"(\d+(?:\.\d+)?)\s*([wdhms])", s.lower())
        if not parts:
            return None
        return sum(float(n) * mult[u] for n, u in parts)
    if units == "seconds":
        return val / 3600.0
    if units == "minutes":
        return val / 60.0
    if units == "days":
        return val * 8.0
    return val


def classify(title, issue_type, labels):
    """Bucket a ticket as novel / routine / standard from its text."""
    blob = " ".join(filter(None, [title, issue_type, labels])).lower()
    if any(m in blob for m in NOVEL_MARKERS):
        return "novel/integration"
    if any(m in blob for m in ROUTINE_MARKERS):
        return "routine/cosmetic"
    return "standard feature/fix"


def bucket_of(hours):
    for name, lo, hi in BUCKETS:
        if lo <= hours <= hi:
            return name
    return ">6h"


def ratio(rows):
    est = sum(r["est"] for r in rows)
    act = sum(r["act"] for r in rows)
    return (act / est) if est else float("nan")


def pct(n, d):
    return (100.0 * n / d) if d else 0.0


def main():
    p = argparse.ArgumentParser(
        description="Derive an estimation baseline from a CSV of resolved issues.")
    p.add_argument("csv_file")
    p.add_argument("--units", default="hours",
                   choices=["hours", "seconds", "minutes", "days"],
                   help="units of the estimate/actual columns (default: hours). "
                        "Jira API exports are usually seconds.")
    p.add_argument("--estimate-col", help="override estimate column name")
    p.add_argument("--actual-col", help="override actual/logged column name")
    p.add_argument("--encoding", default="utf-8-sig")
    p.add_argument("--top", type=int, default=5, help="how many tail outliers to show")
    args = p.parse_args()

    try:
        with open(args.csv_file, newline="", encoding=args.encoding) as fh:
            reader = csv.DictReader(fh)
            fields = reader.fieldnames or []
            raw_rows = list(reader)
    except FileNotFoundError:
        sys.exit(f"error: no such file: {args.csv_file}")

    if not fields:
        sys.exit("error: no header row found in CSV")

    est_col = find_col(fields, ESTIMATE_HINTS, args.estimate_col)
    act_col = find_col(fields, ACTUAL_HINTS, args.actual_col)
    if not est_col or not act_col:
        sys.exit(
            "error: could not identify estimate/actual columns.\n"
            f"  estimate: {est_col or 'NOT FOUND'}\n"
            f"  actual:   {act_col or 'NOT FOUND'}\n"
            f"  available: {', '.join(fields)}\n"
            "Pass --estimate-col / --actual-col explicitly."
        )

    key_col = find_col(fields, KEY_HINTS)
    title_col = find_col(fields, TITLE_HINTS)
    type_col = find_col(fields, TYPE_HINTS)
    label_col = find_col(fields, LABEL_HINTS)

    rows, est_only, act_only = [], 0, 0
    for r in raw_rows:
        e = parse_duration(r.get(est_col), args.units)
        a = parse_duration(r.get(act_col), args.units)
        if e and a and e > 0 and a > 0:
            title = r.get(title_col, "") if title_col else ""
            rows.append({
                "key": r.get(key_col, "?") if key_col else "?",
                "title": title,
                "est": e,
                "act": a,
                "ratio": a / e,
                "class": classify(title,
                                  r.get(type_col, "") if type_col else "",
                                  r.get(label_col, "") if label_col else ""),
            })
        elif e:
            est_only += 1
        elif a:
            act_only += 1

    total = len(raw_rows)
    n = len(rows)
    print("=" * 68)
    print("ESTIMATION BASELINE")
    print("=" * 68)
    print(f"\nSource      : {args.csv_file}")
    print(f"Columns     : estimate={est_col!r}  actual={act_col!r}  (units: {args.units})")
    print(f"Issues      : {total} total")
    print(f"PAIRED (n)  : {n}  <- the real sample size")
    print(f"              {est_only} estimate-only, {act_only} actual-only, "
          f"{total - n - est_only - act_only} with neither")

    if n == 0:
        sys.exit("\nNo paired rows. Check --units and the column overrides.")
    if n < 20:
        print(f"\n  ! n={n} is small. Treat everything below as directional only.")

    # ---- Data quality first: the caveat must be read before the ratio. ----
    within10 = sum(1 for r in rows if 0.9 <= r["ratio"] <= 1.1)
    exact = sum(1 for r in rows if abs(r["ratio"] - 1.0) < 0.005)
    share10 = pct(within10, n)
    print("\n" + "-" * 68)
    print("DATA QUALITY  (read before trusting any ratio)")
    print("-" * 68)
    print(f"Within +/-10% of estimate : {within10}/{n}  ({share10:.0f}%)")
    print(f"Exactly 1.00x             : {exact}/{n}  ({pct(exact, n):.0f}%)")
    if share10 >= 40:
        print("\n  ** BACKFILL DETECTED **")
        print("  Real durations do not cluster this tightly on the predicted value.")
        print("  Logged time here is reconciled to the estimate, not measured.")
        print("  Consequences:")
        print("    - Do NOT quote the median; it measures the convention, not the work.")
        print("    - The aggregate ratio below is a FLOOR, not a central estimate.")
        print("    - The tail is the trustworthy part; weight outliers heavily.")
    else:
        print("\n  Clustering looks plausible; the aggregate ratio is usable as a"
              " central estimate.")

    agg = ratio(rows)
    med = sorted(r["ratio"] for r in rows)[n // 2]
    print("\n" + "-" * 68)
    print("HEADLINE")
    print("-" * 68)
    print(f"Total estimated : {sum(r['est'] for r in rows):.1f}h")
    print(f"Total actual    : {sum(r['act'] for r in rows):.1f}h")
    print(f"AGGREGATE RATIO : {agg:.2f}x"
          + ("   (floor - see data quality)" if share10 >= 40 else ""))
    print(f"Median ratio    : {med:.2f}x"
          + ("   (ignore - backfill artefact)" if share10 >= 40 else ""))

    # ---- Scale actually in use ----
    print("\n" + "-" * 68)
    print("SCALE IN USE  (adopt this, don't impose one)")
    print("-" * 68)
    counts = defaultdict(int)
    for r in rows:
        counts[round(r["est"], 2)] += 1
    for v in sorted(counts):
        bar = "#" * min(40, counts[v])
        print(f"  {v:>6.2f}h  n={counts[v]:<4} {bar}")
    over_cap = sum(1 for r in rows if r["est"] > 6)
    print(f"\n  Above 6h: {over_cap}/{n} ({pct(over_cap, n):.0f}%)")
    if over_cap:
        big = [r for r in rows if r["est"] > 6]
        small = [r for r in rows if r["est"] <= 6]
        if small:
            print(f"  <=6h subset ratio: {ratio(small):.2f}x   "
                  f">6h subset ratio: {ratio(big):.2f}x")
            print("  If the <=6h subset is better behaved, the cap has empirical support.")
    else:
        print("  The team already works under a 6h cap - the rule is descriptive, not imposed.")

    # ---- Size buckets: usually flat, which is the useful finding ----
    print("\n" + "-" * 68)
    print("BY SIZE  (flat ratios => size does not predict overrun)")
    print("-" * 68)
    print(f"  {'bucket':<10} {'n':>4}  {'ratio':>7}")
    for name, lo, hi in BUCKETS:
        sub = [r for r in rows if lo <= r["est"] <= hi]
        if sub:
            print(f"  {name:<10} {len(sub):>4}  {ratio(sub):>6.2f}x")

    # ---- Work class: usually where the signal is ----
    print("\n" + "-" * 68)
    print("BY WORK CLASS  (novelty is the better predictor)")
    print("-" * 68)
    by_class = defaultdict(list)
    for r in rows:
        by_class[r["class"]].append(r)
    print(f"  {'class':<22} {'n':>4}  {'ratio':>7}")
    for cls in sorted(by_class, key=lambda c: -ratio(by_class[c])):
        sub = by_class[cls]
        print(f"  {cls:<22} {len(sub):>4}  {ratio(sub):>6.2f}x")
    if len(by_class) > 1:
        rs = {c: ratio(v) for c, v in by_class.items()}
        spread = max(rs.values()) - min(rs.values())
        if spread >= 0.15:
            print(f"\n  Spread of {spread:.2f}x across classes supports padding by"
                  " UNCERTAINTY rather than size.")
        else:
            print(f"\n  Classes are within {spread:.2f}x of each other - no strong signal."
                  " Fall back to default multipliers rather than inventing a finding.")

    # ---- The tail: the part backfill cannot fake ----
    print("\n" + "-" * 68)
    print(f"TAIL - worst {args.top} by ratio  (go read these)")
    print("-" * 68)
    for r in sorted(rows, key=lambda r: -r["ratio"])[: args.top]:
        print(f"  {r['ratio']:>5.1f}x  {r['est']:>5.1f}h -> {r['act']:>6.1f}h  "
              f"{r['key']:<12} {r['title'][:44]}")
    print(f"\nTAIL - worst {args.top} by absolute hours lost")
    for r in sorted(rows, key=lambda r: -(r["act"] - r["est"]))[: args.top]:
        print(f"  +{r['act'] - r['est']:>5.1f}h  {r['est']:>5.1f}h -> {r['act']:>6.1f}h  "
              f"{r['key']:<12} {r['title'][:44]}")
    print("\n  For each: was it novel work carrying a confident estimate (-> cap + spike),")
    print("  oversized at creation (-> split), or blocked on something external")
    print("  (-> an unnamed prerequisite, which no multiplier fixes)?")

    # ---- Recommendations ----
    print("\n" + "-" * 68)
    print("SUGGESTED RULES")
    print("-" * 68)
    scale = sorted(counts, key=lambda v: -counts[v])[:6]
    print(f"  Scale     : {sorted(v for v in scale if v <= 6)}  (their values, capped at 6h)")
    print("  Max       : 6h  |  Min: 0.5h")
    print("  Cap       : Medium/High uncertainty -> max 2h, else write a 2h spike")
    budget_lo, budget_hi = max(1.3, agg), max(1.5, agg * 1.25)
    print(f"  Sprint    : budget {budget_lo:.1f}-{budget_hi:.1f}x the ticket sum")
    if share10 >= 40:
        print("\n  Improve the baseline: log actuals from a real record instead of")
        print("  reconciling to the estimate afterwards. Two sprints of rough")
        print("  half-hour honesty beats any further analysis of this data.")
    print()


if __name__ == "__main__":
    main()
```

---

*MIT licensed. Free to copy, adapt, and disagree with.*
