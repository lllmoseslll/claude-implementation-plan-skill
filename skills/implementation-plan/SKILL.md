---
name: implementation-plan
description: Implementation plan for one ticket or feature, researched against the latest remote integration branch and written to the project's plans repo. Use when asked to plan a ticket or to write an implementation (execution) plan.
---

# Implementation Plan

Produce a **filled** implementation plan for one ticket or feature, written to the project's
**plans repo** (§3). The template lives beside this file at `template.md`. Read it in full before
starting — it is the authority on section content, and every `>` line in it is an authoring rule.

Below, a bare `§n` is a section of this file; `template §n` is a section of the template and of the
finished plan.

## Who does what: the strongest model plans and reviews, a cheaper model builds

Plans are made by the strongest model available, and each step is built by a cheaper, capable
model, which does well when the task has been defined well first. This skill runs in any agent that
reads `SKILL.md` skills, so it names **roles**, not models:

- **Reviewer**: plans, decides and reviews. It writes the plan (this skill), answers the questions
  that come up during the build, judges findings (a browser walk, a contrast failure, a bug that is
  or is not one) and reviews every step before it is committed.
- **Implementer**: builds one step at a time from the plan's text.

### Pick the models before you write the plan

Resolve each role to a real model once, at the start, and record both in template §0
("Reviewer model" and "Implementer model"). Everywhere else, the plan says *reviewer* and
*implementer*, never a model name, so it stays correct if the models change. The one exception is
the Session log (template §8), which records the models that actually ran each step.

1. **The user or the project says so:** a model named in the request, or in the user's or project's
   agent instructions file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md` or the agent's equivalent), wins.
2. **Otherwise use the default for the agent you are running in:**

   | Agent | Reviewer | Implementer | How the implementer is started |
   |---|---|---|---|
   | Claude Code | The latest Fable; if it is not available, the latest Opus (as a subagent: Agent tool with `model: "fable"`, else `"opus"`) | The latest Sonnet at high effort | Agent tool with `model: "sonnet"`, `effort: "high"` |
   | OpenAI Codex | The strongest model the account offers, at high reasoning effort | A faster model, or the same one at lower effort | A subagent. It uses the model in `.codex/agents/<name>.toml` (`model`, `model_reasoning_effort`) if one is defined; otherwise it inherits the session's model (template §0 then names that model for the implementer). Do not create that file without asking |
   | Google Antigravity | `pro` | `flash` | A subagent via `define_subagent` / `invoke_subagent` with `model` |
   | Any other agent | The strongest model it offers | A cheaper capable model | Its subagent feature, if it has one |

3. **A model is not available** (the account does not offer it, or a call with it is refused):
   use the next strongest one for that role and say once which model ran it.
4. **The session is not on the reviewer model:** hand reviewer work (planning, reviews, hard
   judgments) to a subagent started on the reviewer model, where the agent allows it. Where it does
   not, the session's model does the reviewer work and template §0 records that model.

### How the work is split

- **Implementer: one step per agent,** one agent at a time unless the machine has room for more.
  Its brief is the step itself: the step text, its Verify line, its exact commit message, the
  Context items and Decisions it cites, the files to read, and what not to touch. It builds, runs
  the step's Verify line, and reports. It does not commit, push, decide an open question or change a
  Decision. When it hits one, it stops and reports.
- **No subagents?** If the agent cannot start subagents, it builds each step itself, then switches
  to the reviewer's checks below before committing. The roles stay separate even when one model
  plays both. Both template §0 rows then name the session's model.
- **Every step is reviewed before its commit.** The reviewer reads the whole diff, not the
  implementer's summary, re-runs the step's Verify line itself, checks every claim the implementer
  wrote into code comments and the Session log against the code, and only then commits. Anything
  wrong goes back to the same implementer with the reason (in Claude Code, `SendMessage`; otherwise
  the agent's way of continuing a subagent), or is fixed by the reviewer and noted in the Session log.
- **Risky steps keep the implementer as the builder but get a second opinion:** money, data
  isolation and permissions, migrations and anything that can destroy data, security, production.
  Before that commit, the reviewer asks for a second opinion, using the first of these that exists:
  the agent's advisor tool (Claude Code's `advisor`), a fresh subagent on the reviewer model asked
  to review the diff and advise only, or a second independent review pass of the whole diff. The
  step's template §7 entry is marked "+ second opinion".
- **Stuck:** if the implementer fails the same Verify check twice, the reviewer diagnoses it (with
  a second opinion, chosen as for a risky step) before a third attempt, and either rewrites the brief or
  takes the step over itself. The Session log says which.
- **Because the implementer builds what the plan says, the plan must leave it nothing to decide.**
  Every choice is a Decision in template §3 or an Open question with a gating step; a step that
  needs judgment to build ("make it look right") is split into what the implementer can build and
  what the reviewer judges.

## 1. Get the ticket code

If the invocation carried an argument, that is the ticket code. Otherwise **ask for it before doing
anything else**:

> What's the ticket code for this plan? (e.g. `PROJ-123` or `TASK-42`)

Normalise it to `UPPERCASE-KEY` + number (`proj123`, `proj-123`, `PROJ 123` → `PROJ-123`). If the user gives
a code with no project key, ask for the key.

In the same turn, ask anything the plan cannot be written without and that the repo cannot tell
you — typically just: **what is this ticket actually asking for?**

## 2. Establish the repo facts

If the working directory is not inside a git repo (for example a workspace root that holds several
product repos and a shared `plans/` repo), work out which product repo(s) the ticket touches — from
the request, or ask — and run the commands below **inside each of them**, recording each repo's
facts and branch separately in template §0. The shared plans repo is never a product repo.

**Plans are always based on the latest remote integration branch**, never on a stale local copy.
First identify the integration branch (`develop` if it exists on the remote, otherwise the branch
the repo's agent instructions file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`) or docs name, e.g.
`dev`; never `main` for feature work). Then fetch — a read-only operation, no push — and record the real values; they go in template §0:

```
git rev-parse --show-toplevel
git branch --show-current
git fetch origin                                   # read-only; refresh the remote branches
INT=develop; git rev-parse --verify -q origin/$INT >/dev/null || INT=dev   # adjust if the repo names another
git rev-parse --short origin/$INT                  # the base SHA recorded in template §0
git rev-list --count $INT..origin/$INT             # how far the local copy is behind (report if > 0)
git log --oneline -10 origin/$INT
```

Do all research (§4) against `origin/<integration>` at that SHA: if the local integration branch or
working tree differs from it, read files with `git show origin/<integration>:<path>` or say so in
template §1. If there is no remote, use the local integration branch and say so in template §0.

Derive the branch name as `feature/<TICKET-ID>-<lowercase-hyphen-description>` per template §0.1.
The plan only names the branch; the user creates it. Template §0 must give the exact creation
commands, which always branch from the freshly fetched remote:

```
git fetch origin
git switch -c feature/<TICKET-ID>-<description> origin/<integration>
```

and must tell the executor: if `origin/<integration>` has moved past the SHA the plan was verified
against, list the new commits (`git log --oneline <plan-sha>..origin/<integration>`), re-check every
Context item whose files they touch, and record the new base SHA and any corrections in the Session
log before Step 1.

## 3. Decide where the plan goes

Every project has one **plans repo**: a `plans/` folder that is its own private git repo, shared by
all the project's product repos and kept out of their git (e.g. `my-product/plans/` holding
plans for both `my-product-api/` and `my-product-web/`). Search the working directory and every
parent for it:

```
d=$(pwd); while :; do [ -d "$d/plans/.git" ] && { echo "$d/plans"; break; }; p=$(dirname "$d"); [ "$p" = "$d" ] && break; d=$p; done
```

A location stated in the project's agent instructions file (`CLAUDE.md`, `AGENTS.md`,
`GEMINI.md`; root or repo), e.g. "plans live in `../plans/`", wins over the search.

- **If a plans repo is found, the plan goes there — always**, even when the skill was invoked from
  deep inside one product repo. Leave every product repo as it is: no `plans/` folder created in
  it, its `.git/info/exclude` and `.gitignore` untouched. Say in one line where the plan is going
  and why ("shared plans repo found at …").
- **If none is found, create it at the project root.** The project root is the git top-level of the
  session's primary working directory when that directory is inside a git repo; otherwise the
  primary working directory itself (a workspace folder holding several repos). Never go higher than
  that — a folder above the project (e.g. a general folder holding all your code) would mix
  unrelated projects' plans.

  ```
  root=$(git -C "<primary working directory>" rev-parse --show-toplevel 2>/dev/null || echo "<primary working directory>")
  cd "$root"
  # Refuse if the product repo already tracks plans/ — that repo has its own convention.
  git ls-files plans/ 2>/dev/null | head -1
  mkdir -p plans
  [ -d plans/.git ] || git -C plans init -q -b main
  [ -f plans/.gitattributes ] || printf '*.md text eol=lf\n' > plans/.gitattributes
  # If the root is itself a git repo, keep the nested plans repo out of it — locally, not in .gitignore.
  if git rev-parse --git-dir >/dev/null 2>&1; then
    grep -qxF 'plans/' "$(git rev-parse --git-dir)/info/exclude" || echo 'plans/' >> "$(git rev-parse --git-dir)/info/exclude"
  fi
  ```

  - If `git ls-files plans/` prints anything, **stop and say so** instead of changing how that repo
    handles plans.
  - An existing untracked `plans/` folder is turned into the repo in place; its files stay.
  - Use `.git/info/exclude`, **not** `.gitignore` — `.gitignore` is tracked, and editing it would put
    the very change we are avoiding into the product repo's history.
  - The new plans repo has no remote and is never pushed unless the user asks. Say in one line that
    you created it and where.

Below, `<plans-dir>` means whichever folder this section chose.

## 4. Research before writing

Research by reading the code at the recorded SHA, to the bar of template §1's rules. Research is
done when:

- every Context item cites `path/to/file.ext:line` that you read at that SHA — a bare filename, a
  memory or a spec document is not a citation;
- every grep that returned nothing is recorded as a finding: what you searched for, over which
  trees, and that it returned no match;
- what is already correct is recorded, so no step wastes a commit redoing it;
- every place verification contradicts an existing document is marked `[corrects <document>]`;
- the existing test suite is counted (test classes, integration tests, e2e specs) for the closing
  Context item.

A plan built on three files skimmed is worse than no plan.

## 5. Fill the template

Copy `template.md`'s structure. The plan is filled when:

- every `[MANDATORY]` section is filled and every skipped `[OPTIONAL]` section is deleted;
- every `>` authoring rule has been applied to the section beneath it and then deleted — no `>` line
  survives into the finished plan;
- every plan-content box under "Before the first commit" in the template's Appendix is true: Context
  citations, a rejected alternative per Decision, a gating step per Open question, a Verify line
  and exact commit message per step, every commit message checked against template §0.2, and the
  cut line named in Risks;
- template §0 records the reviewer and implementer models chosen under "Pick the models", and
  template §1 to §7 name no model;
- every step can be built by the implementer from its text alone, and template §7 names, per
  step, the implementer, the reviewer and, for a risky step, the second opinion before commit (see
  "Who does what").

Three things the template leaves to you:

- Template §0's "AI co-author trailers" row is **`none`** unless the project's own rules (its
  agent instructions file or contribution guide) configure trailers — then declare them as configured.
- Template §8 Session log starts empty with just its heading; it is appended per step as the work
  lands, in the same commit as that step's work.
- Where a decision needs an answer you do not have, it is an **Open question** with the
  conservative default and the gating step (template §3), and it goes in the report (§7).

## 6. Write the file

Path: `<plans-dir>/<ticket-id-lowercase>-<short-slug>.md` — e.g. `plans/proj-123-feature-name.md`.
The slug matches the branch description. If the file already exists, ask before overwriting. In a
shared plans repo, the plan's template §0.5 and template §8 say that Session log entries are committed in
the plans repo alongside each step's product commit.

## 7. Report

Tell the user, briefly:

- the plan path, the branch name it assumes, the base SHA it was verified against (from the freshly
  fetched `origin/<integration>`), and whether the local copy was behind;
- the step count and the phase names;
- **every Open question**, called out plainly, with the step each one gates;
- whether the plans repo was found or created this run, that the plan is not committed yet, and
  (when the plans repo sits inside a product repo) that it is excluded via `.git/info/exclude`.

The plan file is the deliverable, and the run ends with it written and nothing else changed: do not
create the branch, do not commit, and do not push — in a shared plans repo too; committing the plan
there follows the user's normal commit-approval rules.
