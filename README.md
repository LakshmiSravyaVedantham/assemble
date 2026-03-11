# Assemble

A Claude Code skill that turns a problem statement into a structured execution organization.

Give it a problem. It assembles the right cross-functional teams, assigns missions, executes work in dependency-based waves using real parallel subagents, and writes artifacts to disk.

```
/assemble
```

---

## What It Does

**Phase 1 — Intake:** A Project Manager asks 4 questions to understand the goal, constraints, and scope.

**Phase 2 — Organize:** PM dynamically selects teams from a library of 8, assigns scoped missions and tasks, groups teams into dependency-based waves, and outputs a full project board for approval.

**Phase 3 — Execute:** PM spawns one real Claude Code subagent per team, in parallel within each wave. Teams write artifacts to `docs/`. PM checkpoints after each wave — you approve before the next wave begins.

**Phase 4 — Close:** PM writes `docs/executive-summary.md` with findings, artifacts, and next steps.

**Querying:** Ask anything at any time — "What is the research team doing?", "Why is infra blocked?", "Show project status" — PM answers from board state, no new agents spawned.

---

## Architecture

```
SKILL.md             ← PM prompt (4 phases + querying) — registered with Claude Code
assemble.md          ← Standalone PM prompt (same content, for reference)
team-agent.md        ← Team agent prompt template
team-library.md      ← 8 default teams with missions and output artifacts
status-schema.md     ← Data contracts between PM and team agents
```

**Pattern:** PM + Flat Team Agents. One Task subagent per team per wave. Wave-based parallel execution — teams with no dependencies run first; dependent teams unlock after wave approval.

**Artifacts written to disk:** Every team writes a real markdown file (`docs/research-notes.md`, `docs/product-spec.md`, etc.). PM writes `docs/executive-summary.md` at close.

---

## Default Teams

| Team | Output Artifact |
|---|---|
| Research | `docs/research-notes.md` |
| Product | `docs/product-spec.md` |
| Design | `docs/ux-flows.md` |
| Engineering | `docs/implementation-plan.md` |
| Infra | `docs/infra-plan.md` |
| QA | `docs/qa-checklist.md` |
| Analysis | `docs/analysis-report.md` |
| Program Management | `docs/risk-register.md` |

PM selects the minimum needed for your project. You can include or exclude teams during intake.

---

## Install

Claude Code skills must be registered via a `SKILL.md` file inside a plugin directory. The support files can live anywhere readable.

**Step 1 — Copy support files:**

```bash
mkdir -p ~/.claude/skills/assemble
git clone https://github.com/LakshmiSravyaVedantham/assemble.git /tmp/assemble
cp /tmp/assemble/team-agent.md ~/.claude/skills/assemble/
cp /tmp/assemble/team-library.md ~/.claude/skills/assemble/
cp /tmp/assemble/status-schema.md ~/.claude/skills/assemble/
```

**Step 2 — Register the skill with Claude Code:**

If you have the [superpowers plugin](https://github.com/obra/superpowers) installed:

```bash
SKILLS_DIR=~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.0/skills
mkdir -p "$SKILLS_DIR/assemble"
cp /tmp/assemble/SKILL.md "$SKILLS_DIR/assemble/SKILL.md"
```

**Step 3 — Run it:**

```
/assemble
```

---

## Example Session

```
/assemble

PM: What are we building or solving?
> Build a CLI tool that analyzes git history and outputs a developer personality report

PM: What constraints matter?
> Python only, no paid APIs, 1 week

PM: What does done look like?
> A pip-installable CLI that outputs a 1-page markdown report in under 5 seconds

PM: Any teams to include or exclude?
> Include Research and Engineering. Skip Design and Infra.

--- ASSEMBLE — Project Board ---
Mission: Build a CLI tool that analyzes git history...
[WAVE 1] Research Team — 3 tasks → docs/research-notes.md
[WAVE 2] Engineering Team (depends on Research) — 4 tasks → docs/implementation-plan.md

Approve? > yes

Wave 1 executing...
✅ Research Team complete — docs/research-notes.md written

Continue to Wave 2? > continue

Wave 2 executing...
✅ Engineering Team complete — docs/implementation-plan.md written

ASSEMBLE — Complete
Artifacts: docs/research-notes.md, docs/implementation-plan.md, docs/executive-summary.md
```

---

## Design Decisions

- **PM never does team-level work.** It organizes, delegates, and reports — never reads or writes project artifacts directly.
- **Wave checkpoints are user-controlled.** After every wave, you see results and decide whether to continue, adjust, or stop.
- **Blocked teams retry once.** If a team fails, PM re-spawns it with adjusted scope. If retry fails, PM surfaces the blocker and waits for your input.
- **Querying is free.** Ask anything at any time — the PM answers from in-memory board state without spawning agents.

---

## License

MIT
