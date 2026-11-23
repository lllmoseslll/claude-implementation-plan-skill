# implementation-plan

A [Claude Code](https://claude.com/claude-code) skill that turns one ticket into a **researched,
verifiable implementation plan**, written to your project's private `plans/` repo.

A plan from this skill is not a to-do list. It is built against the code itself:

- **§0 Contract:** the branch name, the base SHA from a freshly fetched `origin/<integration>`, the
  commit-message format and the PR target.
- **§1 Context:** every claim cites `path/to/file.ext:line`, read at that SHA. A grep that found
  nothing counts as a finding too.
- **§2 Research:** each borrowed pattern or library, and *why it applies here*.
- **§3 Decisions:** numbered, each with a rejected alternative. An open question gets a
  conservative default and a gating step, so the plan never guesses silently.
- **§4 Steps:** one commit per step, each with a **Verify:** line of observable checks and the
  exact commit message.
- **§5 Risks:** includes the cut line, which says where to stop if the plan has to be cut.
- **§6 End-to-end verification, §7 Execution prompts, §8 Session log, §9 PR,** and a pre-flight
  checklist.

It is built for a "strong model plans and reviews, Sonnet builds" workflow. The plan leaves the
implementer nothing to decide, and every step is reviewed before it is committed.

## Install

### As a Claude Code plugin (recommended)

Inside Claude Code:

```
/plugin marketplace add lllmoseslll/claude-implementation-plan
/plugin install implementation-plan@claude-implementation-plan
```

To update later, run `/plugin marketplace update claude-implementation-plan`.

### As a plain skill

Copy the skill folder into your personal skills directory:

```bash
git clone https://github.com/lllmoseslll/claude-implementation-plan.git
mkdir -p ~/.claude/skills
cp -r claude-implementation-plan/skills/implementation-plan ~/.claude/skills/
```

To use it in one project only, copy it into that project's `.claude/skills/` instead.
`SKILL.md` and `template.md` must stay in the same folder.

## Use

Ask Claude Code to plan a ticket, or invoke the skill directly:

```
/implementation-plan PROJ-123
```

The skill:

1. asks for the ticket code (if you didn't give one) and what the ticket is asking for;
2. fetches the latest remote integration branch (`develop`, else `dev`) and records its SHA;
3. finds the project's `plans/` repo by walking up from the working directory. If there is none,
   it creates one at the project root and keeps it out of the product repo with
   `.git/info/exclude`;
4. researches the code at that SHA and fills every mandatory section of `template.md`;
5. writes `plans/<ticket-id>-<slug>.md` and reports the open questions.

It never creates the branch, commits or pushes. Those stay your call.

## Conventions it assumes

The template follows these conventions. Adapt template §0 if your team's differ:

- Branches: `feature/<TICKET-ID>-<lowercase-hyphen-description>`, branched from `origin/develop`.
- Commits: `<TICKET-ID> <Sentence case imperative description>`, under 72 characters, with no
  trailing period.
- One ticket, one branch, one commit per step, one PR.

The model roles (Fable plans and reviews, with Opus as the fallback; Sonnet at high effort builds)
are set in the "Who does what" section of `SKILL.md`. Edit that section to match the models you
have.

## Layout

```
.claude-plugin/
  plugin.json          plugin manifest
  marketplace.json     lets this repo be added as a plugin marketplace
skills/implementation-plan/
  SKILL.md             the skill's instructions
  template.md          the plan template; every ">" line is an authoring rule
```

## Licence

[MIT](LICENSE)
