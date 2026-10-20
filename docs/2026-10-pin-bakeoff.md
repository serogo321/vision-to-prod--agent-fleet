# Pin bakeoff, October 2026: which model and effort each lane should run

Each contract in this fleet pins a model and an effort level. In October 2026 we replayed real past work through the
old pins and candidate new pins, graded the results blind, and changed the pins from what we measured. This is the
method, the results and the final pin table.

## Setup

- **Real cases, replayed.** Every case was a piece of work the fleet had already done, on a project with a known
  answer: the defects that later reviews, later fixes or production found.
- **No peeking.** Every run worked in a plain export of the code as it stood *before* the original work. The export
  had no version-control history, no later reports and no later commits, so a run could not read the answer.
  - A cross-touch check afterwards found 0 runs that opened another run's files.
- **Back-to-back pairs.** For each case the old pin and the new pin ran one after the other on identical inputs. A
  third arm (a cheaper, faster model) ran on the build, research and triage cases.
- **Blind grading.** A separate grader agent saw only arms labelled A / B / C, never which model produced which. It
  scored each arm against the known answer and ranked them.
- **Old pin as dispatched, new pins per call.** The old arm was the real agent dispatched with no parameters, so its
  frontmatter pin applied. New arms set model and effort per dispatch. Every run's transcript was checked for the
  model and effort it actually used.
- **Active time only.** Times are active minutes from the agent transcripts. Silences over 10 minutes (an editor
  crash, overnight and weekend pauses, usage-limit pauses) are excluded. Paused runs were resumed on their own
  transcript after their working trees were checked byte for byte against the originals. None restarted from scratch.
- **Build arms went through review.** Every draft from a build arm went through code review at the new reviewer pin,
  with fix rounds until it passed (cap: 4 rounds). Build time is end to end: the build, every review round and every fix.
- **How "worse" was judged.** A new pin counts as worse when it repeatedly misses a known blocking defect that the
  current pin catches. Defects both pins miss decide nothing. **A known blocking defect that is reported but rated
  non-blocking counts as a miss.**

## The cases

| case | what it was |
|---|---|
| Review case A | A change that adds an age limit to a safety gate, which refuses actions when its cached data is too old. Known blockers: the limit is a fixed number that does not scale with a configurable polling interval, so at a legal setting every action is refused; and dozens of existing tests pass only through shared global state inside a wall-clock window, failing when run alone. |
| Review case B | A new persistence layer for a running counter that feeds an amount calculation. Known blockers: an "unknown" state is stored as a null into a typed field, crashing follow-up hooks after their writes commit; a drift check marks an ordinary re-entry as permanently unknown and refuses later actions; several routes compute an amount larger than allowed; and **an unclamped negative correction that over-counted**, found only by a later review. |
| Review case C | A nine-commit change to an entry path. Known blockers: a helper function defined twice in one module, the later definition silently shadowing the one a calculation depends on and breaking tests that were green before; and an uncontained sweep. |
| Build case D | The fix for review case A, built from its original brief. |
| Build case E | The persistence layer of review case B, built from its design. |
| Research case F | Which configuration setting controls a known blind spot, and what polling floor is safe against an external API's rate limit. Confirmed answer available. |
| Research case G | Does a build platform honour an ignore file, and is a proposed ignore set safe? Confirmed answer: yes and yes. |
| Research case H | A re-study of a vendor API's documentation for a design. The confirmed semantics are what later shipped. |
| Triage case I | Sorting about 100 inbox notes into a backlog: new items, merges into existing items as duplicates, closures. Known answer: 9 new, 11 duplicates, 1 closed, none lost. |

Review cases A and B went to the code reviewer, the critic and the security reviewer. Review case C went to the code
reviewer and the critic.

## Where the time goes

Before testing pins, we measured the last ten merged changes from their transcripts and commit times.

- **Summed lane time:** executor 27%, code review 44%, security review 25%, critic 3%.
- **The largest bucket is no lane at all.** About 3,590 minutes went to gaps (parked work, overnight, waiting on the
  founder) against about 2,258 minutes with at least one lane active.
- **So a faster pin can cut at most the active third of a change's calendar time.** Quality decides; speed breaks ties.

## Results per lane

### Executor: Opus 4.8 high → **Opus 5.5 medium**
| case | arm | first-draft blocking defects | review rounds | end to end (min) | blind rank |
|---|---|---|---|---|---|
| Build case D | Opus 4.8 high | 2 | 2 | 138.6 | 3rd |
| | Opus 5.5 medium | 1 (self-flagged) | 1 | 76.6 | **1st** |
| | Sonnet 5.5 medium | 1, plus a stray file and 3 report errors | 1 | 40.7 | 2nd |
| Build case E | Opus 4.8 high | 2 (one repeats the original build's defect) | 4 (two for defects its own fixes introduced) | 207.7 | 3rd |
| | Opus 5.5 medium | 0 (found and reported a gap in the design itself) | 1 | 74.7 | **1st** |
| | Sonnet 5.5 medium | 0 (same design gap, but wrongly called it safe; 2 report errors) | 4 (its fixes introduced 2 new defects) | 215.6 | 2nd |

- **Quality better:** first-draft blocking defects 4 → 1 over both cases, ranked first both times.
- **Speed:** 346.3 → 151.3 minutes end to end (**−56%**).
- Sonnet tied on blocking count but ranked lower on honesty and clutter, and was slower end to end on case E.

### Researcher: Opus 4.8 high → **Opus 5.5 medium**
| case | Opus 4.8 high | Opus 5.5 medium | Sonnet 5.5 medium |
|---|---|---|---|
| F (run twice) | **wrong on a confirmed answer, both times** | 8/8 both times | no wrong answer; 1 missed, some partial |
| G | 7 right, 1 partial | 8/8 | 8/8 |
| H | 10/10, 2 wrong citations | 10/10, 0 wrong citations | 9/10 + 1 partial; 7 citations off by 1–2 lines |
| blind rank | last, last, 2nd | **1st on all three** | 2nd, 2nd, 3rd |
| total min | 20.8 | 25.5 (≈21.5 without a 4-minute lock wait) | 9.2 |

- **Quality better** (the old pin repeated a wrong confirmed answer). **Speed about equal.**
- Sonnet is far faster but not a quality tie: missed and partial answers, citation drift.

### Product manager (roadmap steward): Opus 5 high → **Opus 5.5 medium**
| arm | wrongly closed | dropped | duplicates missed | write fence | blind rank | min |
|---|---|---|---|---|---|---|
| Opus 5 high | 0 | 0 | 1 soft | kept, but re-derived two protected blocks and claimed them unchanged | 3rd | 39.5 |
| Opus 5.5 medium | 0 | 0 | 1 soft | kept, the only run to copy both protected blocks byte for byte | **1st** | 19.9 |
| Sonnet 5.5 medium | 0 | 0 | 1 soft | **breached** (deleted every inbox file in its copy) | 2nd | 17.3 |

- **Quality better, speed −50%.**

### Code reviewer: Opus 5 xhigh → **Opus 5.5 high**
| case / round | old: blocking defects caught | new: blocking defects caught | blind winner | min old / new |
|---|---|---|---|---|
| A | 2/2 | 2/2 | new | 29.7 / 22.8 |
| B round 1 | 2/6 | 3/6 | new | 30.1 / 23.4 |
| B round 2 | 0/6, **approved** | 2/6 (one at the wrong severity), blocked | new | 27.1 / 23.2 |
| C round 1 | 0/2 | 0/2 | new | 35.7 / 23.0 |
| C round 2 | 0/2 | 0/2 | new | 39.2 / 41.5 |

- The new pin caught every blocking defect the old pin caught. **Quality same or better** on every case.
- **Speed:** 161.8 → 133.9 minutes (**−17%**).
- **Caveat:** reviewing build case D's drafts, the same pin rated a known blocker MEDIUM three times. It is the same
  severity gap the critic shows below.

### Critic: **stays on Opus 5 xhigh** (re-test done: the severity rule didn't close the gap, so the critic stays on Opus 5)
| case / round | Opus 5 xhigh | Opus 5.5 high | blind winner | min old / new |
|---|---|---|---|---|
| A round 1 | **2/2** | 1/2 (**known blocker rated MEDIUM**) | old | 40.7 / 23.3 |
| A round 2 | **2/2** | 1/2 (**rated MEDIUM again**) | old | 44.9 / 26.8 |
| B round 1 | 1 | 2 | new | 31.6 / 31.3 |
| B round 2 | 2 | 3 | new | 30.0 / 17.6 |
| C round 1 | 0/2 | 0/2 (fewer misses) | new | 30.4 / 28.7 |
| C round 2 | 1 at blocking | 0/2 | old | 45.8 / 41.8 |

- Opus 5.5 high **repeatedly missed a known blocker that the old pin caught**, by under-rating it. That counts as
  worse, despite being 24% faster.
- **An added arm, Opus 5.5 xhigh, ran twice on case A:** it rated the known blocker blocking in one run and MEDIUM in
  the other. 1 of 2 does not meet the bar, and it was no faster than the old pin.
- Both xhigh runs did find a real new defect that no other run on case A found.

### Security reviewer: **stays on Fable max**
| case / round | Fable max | Fable xhigh | blind winner | min old / new |
|---|---|---|---|---|
| B round 1 | 1 blocker (+1 at wrong severity) | 1 blocker (+2 at wrong severity) | new (narrow) | 34.2 / 26.1 |
| B round 2 | **2 blockers** (+2 at wrong severity) | 1 blocker; a **false "clean"** on another | **old** | 32.6 / 27.0 |
| A round 1 | 0/2, passed (wrong) | 0/2, passed (wrong) | old (narrow) | 27.4 / 29.7 |
| A round 2 | **1 blocker, blocked** | 0/2, passed (wrong) | **old, clearly** | 32.1 / 22.9 |

- xhigh repeatedly missed a known blocker on both cases that max caught in round 2. **Worse.** It was 16% faster,
  but quality comes first.
- Both pins under-rated one known blocker in every run; those shared misses decide nothing.

### Orchestrator, debugger, test engineer, designer
These lanes can't be replayed as cases. The bakeoff judged them from the reviewer results: at Opus 5.5, **detection**
held up, but **severity judgment** did not (the critic and code-reviewer findings above). These four lanes run on
judgment, so the report recommended no change. **The founder moved them to Opus 5.5 xhigh as a judgment call,** without a
replay, and is watching their rework rounds and rejected-twice counts for a week.

### Architect
Not tested. No change.

## The severity finding

The clearest pattern: **Opus 5.5 finds the defects but sometimes rates a known blocker MEDIUM.** It happened twice to
the critic and three times to the code reviewer on one build case, while Opus 5 xhigh rated the same defect blocking
both times. In review, a blocker rated MEDIUM behaves like a miss: the change merges.

That led to a severity rule, now in the code-reviewer, security-reviewer and critic contracts:

> If a finding can cause a wrong action, a wrong amount, or a safety check that silently stops working, rate it
> blocking. When unsure between medium and blocking, rate it blocking.

With that rule in place, the critic was re-tested on Opus 5.5 xhigh, twice on review case A and graded blind. It
would move to Opus 5.5 xhigh only if both runs rated the known blocker blocking. One run did and one did not, the same
1 of 2 as before the rule. Re-test done: the severity rule didn't close the gap, so the critic stays on Opus 5.

## The checklist lesson

**One defect was missed by every arm in every run:** the unclamped negative correction in review case B, which
over-counted. That covers both code-reviewer pins, both critic pins and both security pins, over two rounds each, plus
all six build drafts of case E and their 16 review rounds. Every run that looked at the correction confirmed it was
"signed and unclamped as designed", the same reasoning error the original reviews made. Only a later review found it.

No pin closed that gap; it is a checklist problem, not a model one. So this line is now in the code-reviewer,
security-reviewer and critic contracts:

> For every quantity computed from a difference or a correction, check what happens when it goes negative or past its
> bound, and require a clamp or a refusal.

## Usage

- **Security model:** 8 runs, 0 refusals, 0 usage-meter failures. One run was cut by a session limit and resumed on its
  own transcript.
- **Tokens, from the bakeoff's own transcripts:**

| model | agents | output tokens | cache read | cache write |
|---|---|---|---|---|
| Opus 5.5 (new arms, every review in the build loop, graders) | 58 | 1.71M | 1,188M | 46.4M |
| Opus 5 (old critic, code-reviewer and product-manager arms) | 13 | 0.65M | 451M | 13.2M |
| Opus 4.8 (old executor and researcher arms) | 10 | 0.40M | 277M | 14.6M |
| Sonnet 5.5 | 8 | 0.15M | 113M | 6.5M |
| Fable 5.1 (security arms) | 8 | 0.51M | 194M | 25.5M |
| main session (Opus 5.5) | 1 | 0.95M | 321M | 5.7M |

## Integrity

- 0 runs touched another run's files.
- One fence breach (the Sonnet triage arm deleted the inbox inside its own copy), counted against it. One flagged
  breach was a false positive: a word in the run's own prose, not a path it opened.
- No writes to the project, no deploys, no calls to any live system.

## Final pin table

| agent | model | effort | basis |
|---|---|---|---|
| `executor` | claude-opus-5-5 | medium | measured: better and −56% |
| `researcher` | claude-opus-5-5 | medium | measured: better, equal speed |
| `product-manager` | claude-opus-5-5 | medium | measured: better and −50% |
| `code-reviewer` | claude-opus-5-5 | high | measured: same or better and −17% |
| `project-manager` | claude-opus-5-5 | xhigh | founder decision, not replayed; under watch |
| `debugger` | claude-opus-5-5 | xhigh | founder decision, not replayed; under watch |
| `test-engineer` | claude-opus-5-5 | xhigh | founder decision, not replayed; under watch |
| `designer` | claude-opus-5-5 | xhigh | founder decision, not replayed; under watch |
| `critic` | claude-opus-5 | xhigh | measured: Opus 5.5 under-rated a known blocker; re-test done: the severity rule didn't close the gap, so the critic stays on Opus 5 |
| `security-reviewer` | claude-fable-5-1 | max | measured: xhigh worse |
| `architect` | claude-fable-5-1 | xhigh | not tested |

Two method notes for anyone repeating this:
- **Check what each run actually used.** An inline `model` alias resolves to whatever that alias means on your build,
  and an edited definition may not reach new launches until the session restarts. Verify from each run's transcript,
  never the UI.
- **Grade severity, not just detection.** Counting a blocker reported at the wrong severity as caught would have hidden
  the main finding.
