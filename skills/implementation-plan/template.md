# \<Plan title — what this branch closes, in one line\>

> **How to use this template.** Copy it to `plans/<ticket-id-lowercase>-<short-slug>.md`, e.g.
> `plans/proj-123-close-the-gap.md`. Every line beginning with `>` is an authoring rule — **delete
> each one as you fill the section beneath it.** Sections are marked **[MANDATORY]** or
> **[OPTIONAL]**; an optional section that you skip is deleted, not left empty with "N/A".
>
> This template assumes **one parent ticket, one branch, one commit per step, one pull request** —
> see §0. It is written to satisfy standard branch-name and commit-message conventions.
> Where team-specific guidelines differ, adapt §0 accordingly.

\<Date\> — plan for branch `feature/<TICKET-ID>-<lowercase-hyphen-description>`, based on
`develop@<short-sha>`. \<One or two sentences: what body of specified-but-unbuilt work this takes,
where that work is recorded, and that it turns it into an ordered, verifiable build.\> Format follows
a standard execution plan: verified Context with `file:line`, a Research section
that says why each borrowed pattern applies *here*, numbered Decisions naming rejected alternatives,
one commit per step with a Verify line, then Risks, end-to-end Verification, per-step Execution
prompts with model selection, and a Session log appended per step.

---

## 0. Branch, ticket and commit contract **[MANDATORY]**

> Fill this table first. It is the part a reviewer and a ruleset both check, and it is the part that
> makes a single-branch plan structured and clean.

| Field | Value |
|---|---|
| Parent ticket | `<TICKET-ID>` — e.g. `PROJ-123` (uppercase project key, ticket number) |
| Jira / Issue status at plan approval | In Progress |
| Branch | `feature/<TICKET-ID>-<lowercase-hyphen-description>` |
| Branched from | `origin/develop` @ `<short-sha>` — the latest remote integration branch, fetched when the plan was written |
| Branch creation | `git fetch origin && git switch -c feature/<TICKET-ID>-<description> origin/develop` — always from the freshly fetched remote, never a local branch |
| PR target | `develop` |
| Commit message format | `<TICKET-ID> <Sentence case imperative description>` |
| Steps in this plan | `<n>` — one commit each |
| Reviewers | `<at least one team member — not the author>` |
| AI co-author trailers | `<none | as configured>` — declare it once here, apply it to every commit |

### 0.1 Branch rules (ruleset-enforced)

- Type is one of `feature`, `hotfix`, `release`. An execution plan branched from `develop` is
  **`feature`** unless it is an urgent fix off `main`, which is **`hotfix`** and does not use this
  template's multi-step shape.
- Ticket ID is the uppercase project key plus the ticket number, matching your issue tracker: `PROJ-123`.
- Description is lowercase and hyphen-separated — no spaces, underscores or special characters — and
  two to five words.
- `main` and `develop` are exempt and must never be prefixed.
- **Never commit directly to `main` or `develop`.**
- **Always branch from the latest `origin/develop`**: `git fetch origin` first, then
  `git switch -c <branch> origin/develop`. If `origin/develop` has moved past the SHA this plan was
  verified against, list the new commits (`git log --oneline <plan-sha>..origin/develop`), re-check
  every Context item whose files they touch, and record the new base SHA in the Session log before
  Step 1.

| Example | Valid |
|---|---|
| `feature/PROJ-123-close-gap` | ✅ |
| `feature/PROJ-123-add-settings` | ✅ |
| `hotfix/PROJ-99-fix-null-pointer` | ✅ |
| `feature/close-gap` | ❌ missing ticket ID |
| `PROJ-123-close-gap` | ❌ missing type prefix |
| `Feature/PROJ-123-close-gap` | ❌ type must be lowercase |
| `feature/PROJ-123-Close-Gap` | ❌ description must be lowercase |

### 0.2 Commit rules (ruleset-enforced)

- Format is `<TICKET-ID> <description>`, ticket ID uppercase.
- Description in **sentence case** — first word capitalised only.
- **Imperative mood**: `Add`, `Fix`, `Update` — never `Added`, `Fixed`, `Updated`.
- Under 72 characters total. No trailing period.

| Example | Valid |
|---|---|
| `PROJ-123 Add rate limiting` | ✅ |
| `PROJ-123 Fix error body on unhandled exceptions` | ✅ |
| `proj-123 add rate limiting` | ❌ lowercase ticket ID |
| `PROJ-123 Added rate limiting.` | ❌ past tense and trailing period |
| `PROJ-123` | ❌ no description |
| `api/rate-limits: per-tenant limits` | ❌ no ticket ID; not the enforced format |

### 0.3 Area tags **[OPTIONAL]**

> If your plan spans several concerns, give each an area tag and use it in **step headings and the
> Session log only** — never in the commit message, which has one legal shape (§0.2). Declare the
> tags here so a reader can group seventeen commits by concern without them.

| Tag | Covers |
|---|---|
| `<Tag>` | `<concern>` |

### 0.4 Where this differs from plans you may have seen

> Keep this subsection. It is the one place the template explains itself, and it stops a reviewer
> re-litigating a resolved conflict. Rewrite the example below to name the shape your repo's older
> plans or commits actually used, or drop that paragraph if there is none.

Older plans in this repo used commit prefixes of the form `api/rate-limits: …`. That shape
**does not pass the commit ruleset** in §0.2. In this template the concern moves to an area tag
(§0.3) carried by the step heading, and the commit message is always
`<TICKET-ID> <Sentence case description>`. Nothing else about those plans' structure changes.

Most contribution guides also ask for small, focused PRs — one ticket per PR. A single-branch plan of many steps is
**one ticket and many commits**, which satisfies that rule; if the plan grows past what one reviewer
can hold, cut steps from the end (see Risks) rather than splitting the ticket mid-flight.

### 0.5 What every step's commit contains **[MANDATORY]**

> Adjust the list to this repo, then treat it as non-negotiable. A step that lands without these is
> not done.

Every step's single commit includes, alongside the code:

1. The step's tests — added, not assumed.
2. `RUNNING.md` entry (or this repo's equivalent running log).
3. Any status table or overview document the step makes stale, updated in the **same** commit.
4. This plan's Session log entry for the step (§8).

> Why in the same commit: a documentation sweep at the end recovers none of what was learned
> mid-step, and a plan that lets its own reference docs go stale re-creates the problem it was
> written from.

---

## 1. Context (verified against `develop@<short-sha>`, \<date\>) **[MANDATORY]**

> **Rules for this section.**
> - Every claim is *verified against the code at the named commit*, not from memory or from a
>   specification document. Cite `path/to/file.ext:line` — a bare filename is not a citation.
> - A grep that returns nothing is a finding: say what you grepped for, across which trees, and that
>   it returned no match.
> - Number the items. Steps, Decisions and Risks all refer to them by number.
> - When verification contradicts an existing document, mark the item
>   **[corrects \<document\>]** and keep it here as Context, not as an errata note — it changes what
>   a step has to do.
> - Record things that are **already correct** too, so no step spends a commit re-doing them.
> - Do not soften a finding. "The screen a new user sees is an apology" is more useful to a reviewer
>   than "onboarding is incomplete".

1. **\<One-line claim, bolded.\>** \<The evidence: file, line, what it actually does. What follows
   from it — which requirement is unbuilt, which behaviour is unreachable, what a user actually
   experiences today.\>

2. **\<Next claim.\>** \<Evidence. Consequence.\>

3. **[corrects \<document\> §\<n\>] \<Claim.\>** \<Evidence. Why the existing document is wrong, and
   what is true instead.\>

\<…\>

*n.* **The test suite is the safety net every step below leans on.** \<Counts: backend test classes,
integration tests over real migrations, boundary/architecture tests, frontend unit files, e2e specs.
Then:\> **Every step in this plan adds to it rather than assuming it.**

---

## 2. Research: what the borrowed patterns are, and why each applies here **[MANDATORY]**

> **Rules for this section.**
> - One numbered entry per external pattern, library, licence or approach the plan leans on.
> - Each entry has two halves: what the source actually says (linked, with version numbers where
>   they matter), and a bolded **Why it applies here:** that ties it to a numbered Context item.
> - A link without the "why it applies here" half is not research, it is a bookmark.
> - Terms and licences count as research. If third-party data or an API has usage terms, quote the
>   binding sentence.
> - Close with **What this research does not justify** — the adjacent things a reader might assume
>   are in scope and are not. This is the cheapest way to stop scope creep at review.

**1. \<The finding, as a claim.\>** \<What the source says, with links and versions.\>
**Why it applies here:** \<Context \<n\> shows \<x\>; this pattern fits because \<y\>; the obvious
alternative fails because \<z\> in *this* codebase specifically.\>

**2. \<…\>**

**What this research does not justify.** \<A list of adjacent things this plan is not licensed to
build by the above.\>

---

## 3. Decisions **[MANDATORY]**

> **Rules for this section.**
> - Number them `D1`, `D2`, … Steps cite them; the Session log cites them when it departs from them.
> - Each decision states the choice in the heading, then the mechanism in a sentence or two, then
>   ***Rejected:*** each alternative **with the reason it was rejected for this system** — not a
>   general-purpose downside.
> - A decision with no rejected alternative is a description, not a decision. Find the alternative
>   or fold the text into a Step.
> - Where a decision depends on a product or business answer you do not have, write
>   **Open question, flagged rather than assumed:** state the conservative default the plan will take
>   and the step that must not start before it is answered. Never silently pick one.

**D1 — \<The decision, as an imperative claim.\>** \<Mechanism.\> *Rejected:* \<alternative\> —
\<why, here\>. *Also rejected:* \<alternative\> — \<why, here\>.

**D2 — \<…\>** \<…\> **Open question, flagged rather than assumed:** \<the question\>. This plan
\<the conservative default\> and asks for a decision before Step \<n\>; the alternative is
\<consequence\>.

---

## 4. Steps **[MANDATORY]**

> **Rules for this section.**
> - **One step = one commit.** If a step needs two commits it is two steps.
> - Group steps into lettered Phases whose names say *why that group is grouped* — "Debt that gets
>   worse with real traffic" earns its heading; "Backend work" does not.
> - Order by what unblocks the next real milestone, not by document order. Say so in a Decision.
> - Every step heading carries its Decision and requirement references: `(D4, D5; FR-04 – FR-09)`.
> - Every step ends with a bolded **Verify:** line — a semicolon-separated list of *observable*
>   checks. "Works correctly" is not a check. Name the test, the assertion, the number.
> - Where the point of a step is an isolation or safety boundary, say that the **negative test is the
>   deliverable** and the feature is incidental.
> - Where a step must not start before an Open question is answered, put
>   **Confirm \<Dn\>'s open question before starting.** in bold in the step body.
> - **Write each step so Sonnet can build it from the text alone** (it implements every step;
>   the planner and reviewer are Fable, else Opus). Name the files, the functions, the exact
>   behaviour, the tests with their negatives, and what not to touch. A choice belongs in §3, not in
>   the step. A step that needs judgment to build is split: what Sonnet builds, then what the
>   reviewer judges.
> - Steps are numbered once and never renumbered. If a step is added late, it takes the next free
>   number and sits in its logical phase — and its Session log entry opens by explaining why the
>   number is out of sequence.

### Phase A — \<Why these are grouped, and why first\>

#### Step 1 — \<What it does\> (\<Dn\>; \<requirement refs\>).
\<What changes, in the imperative, naming the files, endpoints, columns, migrations and screens. Two
to six sentences. Include the shape of any new error, header or response the step introduces, and any
classification change on the client that the step's own change makes necessary.\>
**Verify:** \<observable check\>; \<observable check\>; \<a negative check\>; \<the quality-gate
commands this repo uses, green\>.

**Commit:** `<TICKET-ID> <Sentence case description>`

#### Step 2 — \<…\> (\<Dn\>; \<refs\>).
\<…\>
**Verify:** \<…\>

**Commit:** `<TICKET-ID> <Sentence case description>`

### Phase B — \<…\>

\<…\>

---

## 5. Risks **[MANDATORY]**

> **Rules for this section.**
> - Numbered. Each risk names the **step** it attaches to and the **failure it would actually
>   produce** — a symptom someone would see, not a category.
> - State the mitigation and whether it is already in a step's Verify line. If a risk is accepted
>   unmitigated, say so plainly and say why.
> - Include ordering risks: two steps that touch the same code path, and which must land first.
> - Include the "if the plan is cut, cut from here" risk. A plan of this size will be cut, and
>   deciding the cut line now is better than deciding it under pressure.
> - Carry forward any known risk from a previous plan that this one does not address, marked as
>   unchanged and unaddressed.

1. **\<Step \<n\> can \<the concrete bad outcome\>.\>** \<Why. What the mitigation is. Which Verify
   line tests it. If it is the single most damaging thing in the plan to get wrong, say so.\>
2. **\<Ordering risk.\>** Land Step \<a\> first, run the full suite, then Step \<b\> — \<why a
   combined change bisects badly\>.
3. **\<Open-question risk.\>** **Confirm before Step \<n\>.**
4. **\<n\> steps is \<a quarter of work, not a sprint\>.** \<Which phases are independently
   shippable.\> If the plan is cut, cut from the end.
5. **\<Known risk carried forward, unchanged and unaddressed here.\>**

---

## 6. Verification (end-to-end, after Step \<n\>) **[MANDATORY]**

> **Rules for this section.**
> - This is the acceptance test for the *whole branch*, run before the PR is opened — not a summary
>   of the per-step Verify lines.
> - Each item is a user-visible or API-observable journey that crosses steps.
> - Include at least one cross-tenant / cross-actor negative journey.
> - Close with the quality gates by name, plus any architecture or privilege tests that must still
>   pass, plus the documentation state the branch must leave behind.

1. \<A full journey across several steps, with the figure or elapsed time it must produce.\>
2. \<A configuration journey: everything an owner can now change, taking effect with no deploy.\>
3. \<A reconciliation: filtered figures summing to unfiltered totals.\>
4. \<An arithmetic check against a hand-computed expectation.\>
5. \<A permissions check asserted through the API, not the UI.\>
6. \<The negative isolation journey.\>
7. \<A degradation check: the dependency unreachable, the app still working.\>
8. \<Quality gates\> green; \<architecture/privilege tests\> still pass; \<status docs\> show no
   remaining "not yet built" row except the deliberate exclusions.

---

## 7. Execution prompts **[MANDATORY]**

> **Rules for this section.**
> - One numbered entry per step, in landing order — this is the list someone works down.
> - Format: Sonnet (high) → **reviewer** · `<TICKET-ID> <Sentence case commit message>` — Step
>   \<n\> (\<what the reviewer checks, usually a Risk reference\>).
> - **Every step is implemented by a Sonnet subagent at high effort** (`model: "sonnet"`,
>   `effort: "high"`), briefed with the step, its Verify line, its commit message and the Context
>   and Decisions it cites. It does not commit, push or decide.
> - **The reviewer is the latest Fable, else the latest Opus.** It reads the whole diff, re-runs the
>   Verify line, checks the agent's claims against the code, and commits.
> - **Mark a risky step "+ advisor"**: stock or money arithmetic, isolation and permission
>   boundaries, migrations and anything that can silently destroy data, security, defaults that
>   contradict a specification, production. Sonnet still builds it; the reviewer consults the
>   advisor (or a Fable/Opus helper) before committing, and the parenthetical says what to check.
> - A step that is judgment rather than building (a browser walk against designs, a review, a
>   release decision) says which part Sonnet runs and which part the reviewer decides.
> - Where a step is gated on an answer, put **confirm \<x\> first** in bold in the parenthetical.
> - The commit message here is the *exact* string that will be committed. Check it against §0.2.

1. Sonnet (high) → **Fable / Opus + advisor** · `<TICKET-ID> <Sentence case description>` — Step 1
   (\<what the review must prove\>).
2. Sonnet (high) → Fable / Opus · `<TICKET-ID> <Sentence case description>` — Step 2 (\<…\>).
3. \<…\>

---

## 8. Session log **[MANDATORY]**

> **Rules for this section.**
> - Appended **per step, in the same commit as that step's work** — never as a sweep at the end.
> - This is the most valuable section in the file. It is where the plan is corrected by contact with
>   the code. Write what the plan got wrong, in bold, first.
> - Each entry: bolded findings as claims, then a **Verified:** block listing the tests by name and
>   count, with before/after totals so the delta is checkable.
> - **Record departures from the plan explicitly.** If a Decision's *mechanism* turned out to be
>   impossible, say so, say what replaced it, and say whether the Decision's *intent* survived.
> - Say who built and who reviewed the step (e.g. "Sonnet 5.5 high; reviewed by Fable 5.2"), what
>   the review changed or sent back, and whether the advisor was consulted.
> - Record work done that the plan did not ask for, marked as such.
> - Record environment traps that will bite the next person (toolchain versions, a red suite that is
>   not a real failure, a service that must be running).
> - Never rewrite an earlier entry to match a later understanding. Append.

### Step \<n\> — `<TICKET-ID> <Sentence case description>`

> If this step's number is out of sequence, open with why.

**\<What the plan said, and why it was wrong.\>** \<The evidence. What replaced it. Whether the
governing Decision's intent holds unchanged and only the mechanism differs.\>

**\<A design question the plan left open, and how the code answered it.\>** \<…\>

**\<Work done that the plan did not ask for, marked as unasked.\>** \<Why leaving it out would have
been worse.\>

**\<Environment trap.\>** \<The exact symptom, the exact cause, the exact fix. Include it even if it
is already documented elsewhere — the next person reads this file.\>

**Verified** (\<the command, the toolchain version, any service that had to be running\>):
1. `<TestClassName>` (\<n\> tests): \<what they assert, including every negative case\>.
2. `<IntegrationTestName>` (new, \<n\> tests, real migrations, real tokens): \<the plan's own Verify
   clause, literally — narrate the journey with the actual figures asserted\>.
3. \<Quality gate\> \<after\>/\<after\> green (\<before\> before this step — so the delta is
   +\<n\>: \<the breakdown\>).
4. \<Architecture/boundary test\> clean — \<what boundary it confirms\>.

**\<Scope note: what was deliberately not expanded, and which Decision bounds it.\>** \<…\>

---

## 9. Pull request and review **[MANDATORY]**

> Fill this in when you open the PR. Do not open it before §6 is green.

- **Title:** `<TICKET-ID> <Sentence case description>` — same shape as a commit message (§0.2).
- **Description:** what changed and why, the phases covered, and a link to this plan file.
- **Jira:** PR linked to `<TICKET-ID>`; ticket moved out of In Progress.
- **Reviewers:** at least one team member. **Do not merge your own pull request.**
- **As author:** respond to every comment before requesting re-review; do not resolve reviewer
  comments yourself; if you disagree, discuss it rather than silently ignoring it.
- **As reviewer:** review within one business day where possible; approve only when genuinely
  satisfied; label non-blocking points clearly, e.g. `nit:`.
- **Merge:** only once approved and all automated checks pass.

---

## Appendix — Pre-flight checklist

> Run down this list before the first commit, and again before opening the PR. Delete the appendix
> if your team keeps it elsewhere.

**Before the first commit**

- [ ] Ticket exists in Jira and is In Progress
- [ ] Branch name matches `feature/<TICKET-ID>-<lowercase-hyphen-description>` (§0.1)
- [ ] `git fetch origin` run; branch created from `origin/develop`, not a local branch
- [ ] If `origin/develop` moved past the plan's SHA: new commits listed, affected Context re-checked,
      new base SHA recorded in the Session log
- [ ] Context items all carry `file:line` verified at that SHA
- [ ] Every Decision names at least one rejected alternative
- [ ] Every Open question has a named gating step, and the gating step says so
- [ ] Every step has a Verify line and an exact commit message
- [ ] Every commit message in §7 checked against §0.2 (uppercase ticket, sentence case, imperative,
      under 72 chars, no period)
- [ ] Risks name the cut line

**Per step, before committing**

- [ ] Built by a Sonnet (high) subagent from the step's text; anything it could not build without
      deciding came back as a question, not a guess
- [ ] Reviewed by Fable (else Opus): the whole diff read, the agent's claims checked against the code;
      a risky step ("+ advisor" in §7) also reviewed with the advisor
- [ ] Tests added, not assumed
- [ ] The step's own Verify line actually run by the reviewer, not reasoned about
- [ ] `RUNNING.md` and any status docs updated in this same commit
- [ ] Session log entry written in this same commit, including departures from the plan
- [ ] Commit message is the exact string from §7
- [ ] Nothing committed directly to `main` or `develop`

**Before the PR**

- [ ] Latest `origin/develop` merged or rebased in (`git fetch origin` first) and the suite re-run
- [ ] §6 end-to-end Verification green in full
- [ ] Quality gates green on the branch head
- [ ] Session log has an entry for every landed step
- [ ] Steps not built are marked paused with a reason, not silently absent
- [ ] PR title, description, Jira link and reviewer set per §9
