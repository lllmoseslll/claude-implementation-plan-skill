# implementation-plan

An [Agent Skill](https://agentskills.io) that turns one ticket into a **researched, verifiable
implementation plan**, written to your project's private `plans/` repo. It was built for
[Claude Code](https://claude.com/claude-code), and it installs in Codex, Antigravity, Gemini CLI,
Cursor, GitHub Copilot, OpenCode and any other agent that reads `SKILL.md` skills.

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

Pick the agent you use. Each one only needs the `skills/implementation-plan/` folder, with
`SKILL.md` and `template.md` kept together and the folder still named `implementation-plan`.

### Any agent: one command (`npx skills`)

The [`skills`](https://github.com/vercel-labs/skills) CLI installs the skill for one or more
agents. It needs Node.js.

```bash
# Into the current project, for the agents you name
npx skills add lllmoseslll/claude-implementation-plan-skill -a codex -a antigravity

# For every project on your machine (user scope)
npx skills add lllmoseslll/claude-implementation-plan-skill -a codex -g

# See what it would install without installing
npx skills add lllmoseslll/claude-implementation-plan-skill -l
```

Common agent names for `-a`: `claude-code`, `codex`, `antigravity`, `gemini-cli`, `cursor`,
`github-copilot` and `opencode`. Add `--copy` to copy files instead of symlinking them, which is
safer on Windows. The CLI sends the repo and skill name as anonymous usage stats; set
`DISABLE_TELEMETRY=1` to turn that off.

### Claude Code

As a plugin (recommended), inside Claude Code:

```
/plugin marketplace add lllmoseslll/claude-implementation-plan-skill
/plugin install implementation-plan@claude-implementation-plan-skill
```

To update later, run `/plugin marketplace update claude-implementation-plan-skill`.

Or copy the folder in by hand:

```bash
git clone https://github.com/lllmoseslll/claude-implementation-plan-skill.git
mkdir -p ~/.claude/skills
cp -r claude-implementation-plan-skill/skills/implementation-plan ~/.claude/skills/
```

To use it in one project only, copy it into that project's `.claude/skills/` instead.

### OpenAI Codex

```bash
npx skills add lllmoseslll/claude-implementation-plan-skill -a codex -g
```

Or copy the folder by hand. Codex reads `~/.agents/skills/` for all projects and `.agents/skills/`
inside a project:

```bash
git clone https://github.com/lllmoseslll/claude-implementation-plan-skill.git
mkdir -p ~/.agents/skills
cp -r claude-implementation-plan-skill/skills/implementation-plan ~/.agents/skills/
```

### Google Antigravity

```bash
npx skills add lllmoseslll/claude-implementation-plan-skill -a antigravity -g
```

Or copy the folder by hand. Antigravity reads `~/.gemini/config/skills/` for all projects and
`.agents/skills/` inside a project. Run `/skills reload` afterwards if it was already open.

```bash
git clone https://github.com/lllmoseslll/claude-implementation-plan-skill.git
mkdir -p ~/.gemini/config/skills
cp -r claude-implementation-plan-skill/skills/implementation-plan ~/.gemini/config/skills/
```

### Gemini CLI

```bash
gemini skills install https://github.com/lllmoseslll/claude-implementation-plan-skill.git \
  --path skills/implementation-plan --scope user
```

Or copy the folder into `~/.gemini/skills/` (all projects) or `.gemini/skills/` (one project).

### GitHub Copilot

```bash
gh skill install lllmoseslll/claude-implementation-plan-skill implementation-plan --scope user
```

Or copy the folder into `~/.copilot/skills/` (all projects) or `.github/skills/` (one project).

### Cursor and OpenCode

Use `npx skills add … -a cursor` or `-a opencode`, or copy the folder by hand:

| Agent | All projects | One project |
|---|---|---|
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `.opencode/skills/` |

### Tip: one folder for every agent in a project

Every agent above also reads `.agents/skills/` inside a project. If your team uses several agents,
put the skill there once and commit it with the project:

```bash
mkdir -p .agents/skills
cp -r claude-implementation-plan-skill/skills/implementation-plan .agents/skills/
```

## Use

Ask your agent to plan a ticket, for example "write an implementation plan for PROJ-123". Agents
pick the skill up from its description. You can also start it by name:

| Agent | Command |
|---|---|
| Claude Code | `/implementation-plan PROJ-123` |
| Codex | `$implementation-plan PROJ-123`, or pick it from `/skills` |
| Others | ask for an implementation plan, or use the agent's skills menu |

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
are set in the "Who does what" section of `SKILL.md`, written in Claude Code terms. In another
agent, read them as roles: your strongest model plans and reviews, and a cheaper capable model (a
subagent, if your agent has them) builds one step at a time. If your agent has no subagents, it
builds each step itself, then reviews the whole diff and re-runs the step's Verify line before
committing. Edit that section to match the models you have.

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
