# Vision to prod: an agent fleet for Claude Code

Eleven [Claude Code](https://docs.claude.com/en/docs/claude-code) subagent contracts that work as one team:
- **an orchestrator** that plans and routes the work;
- **specialists** that design, build, test, debug and research;
- **independent reviewers** that never approve their own work.

Every rule in these contracts came from a failure that cost something real. They are opinionated on purpose.

Install: hand [INSTALL.md](INSTALL.md) to Claude Code.

New to it? Read [HOW-TO-USE.md](HOW-TO-USE.md) next. It explains in plain words what to type, how to write a good
milestone, and gives simple and complex examples.

## The agents

The model and effort pins come from a blind bakeoff on real replayed work: see
[docs/2026-10-pin-bakeoff.md](docs/2026-10-pin-bakeoff.md) for the method, results and reasons.

| Agent | Model · effort | What it does |
|---|---|---|
| `project-manager` | Opus 5.5 · xhigh | The orchestrator. It orients from the project's own state files, routes each task to the right specialist, verifies what comes back, and owns the end-of-session state update. |
| `architect` | Fable · xhigh | Designs: systems, fixes, migration and refactor plans, API and contract shapes. It writes a design artifact and never edits source. |
| `critic` | Opus 5 · xhigh | Adversarial review of plans, designs and claims before anyone acts on them. It refutes with evidence or approves with a verified-clean list. |
| `executor` | Opus 5.5 · medium | Implementation. It lands a reviewed plan, verifies with mutations and real exit codes, and never invents scope. |
| `test-engineer` | Opus 5.5 · xhigh | Test strategy, integration and end-to-end coverage, flaky-test hardening, and proof that a test is not vacuous. |
| `debugger` | Opus 5.5 · xhigh | Root-cause analysis. It reads the logs first, proves the mechanism, then fixes minimally. |
| `code-reviewer` | Opus 5.5 · high | The last gate before merge: logic defects, cross-layer drift, fail-open branches, test pinning. Findings carry severity and confidence; it never edits. |
| `security-reviewer` | Fable · max | Security review: auth and sessions, secrets, input parsing, data access, spend paths, new endpoints, dependencies and supply chain. |
| `designer` | Opus 5.5 · xhigh | User-facing surfaces: flows, state inventories, tokens, and visual and interaction specs, backed by a measured prototype. |
| `researcher` | Opus 5.5 · medium | Codebase exploration, library and SDK grounding, external docs. It returns findings and authors nothing. |
| `product-manager` | Opus 5.5 · medium | Steward of the roadmap. It drains the intake inbox, classifies duplicates, closes items with evidence, and regenerates the roadmap view. |

## How they work together

- **The main session hands multi-step work to `project-manager`,** which dispatches the specialists. A single bounded
  task can go straight to one specialist.
- **Authoring and review are always separate agents.**
  - The typical chain: `architect` designs, `critic` attacks the design, `executor` builds, `test-engineer` proves the
    tests bite, and `code-reviewer` plus `security-reviewer` gate the merge.
  - Nobody approves their own work.
- **One writer for the roadmap.** Only `product-manager` edits the roadmap canon. Every other agent files findings into
  the roadmap inbox.
- **Model and effort are pinned in each contract's frontmatter.**
  - Dispatch these agents with no inline `model` parameter. An inline model overrides the pin.
  - If your account lacks a pinned model, edit that file's `model:` line.
- **Reports come back as each agent's final message.** The review and design lanes also write their full report to
  a file, so a crashed lane loses nothing.

## Optional tools and skills

Nothing here is required. Each contract uses these **only if available**.

- **Tool servers (MCP):**
  - `context7`, for library and SDK docs (most agents);
  - `exa` and `firecrawl`, for web search and extraction (`researcher`);
  - `playwright`, for browser automation (`test-engineer`).
- **Skills (public, optional, if installed):**
  - `investigate`, `careful`, `guard`, `cso`, `qa`, `retro`, `context-save` / `context-restore`, `learn`, `scrape`;
  - `playwright-cli`, `example-skills:webapp-testing`, `browse` (`test-engineer`); `codex`, `plan-eng-review`,
    `plan-ceo-review`, `plan-tune` (`critic`);
  - `api-designer`, `rag-architect`, `deployment-patterns`, `vercel-composition-patterns`,
    `vercel-react-best-practices`;
  - `security-review`, `claude-api`, `dataviz`, `ui-ux-pro-max`.
- **Skills named only to ban them.** Some contracts name a skill only to forbid it in that lane, for example skills that
  spawn their own agent fleets or write to the repo from a review lane (`simplify`, `ship`, and `browse` in
  the designer's lane). A ban needs nothing installed.

## Files the fleet expects or creates

The agents look for these in the project they work on, and use what exists. None is required.

- **Roadmap:**
  - `ROADMAP.md` (a generated view, never hand-edited);
  - `roadmap/items/<PREFIX>-####.md` (the canonical items);
  - `roadmap/inbox/` (intake: agents add files and never edit them);
  - the roadmap's own configuration file, and a roadmap view generator and validator if the project has them
    (`product-manager` runs them and never writes them).
- **State and decisions:** `BUILD_STATE.md`, `STATUS.md`, `*_project_state.md`, `TODO.md`, `DECISION_LOG.md`,
  `decisions.md`, `docs/decisions/`, `docs/adr/`, `docs/archive/`.
- **Constitution and conventions:** `README.md`, `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `docs/spec/`, and the
  project's engineering rules or lessons file (`lessons.md`), if it has one.
- **Created by the agents:**
  - the review and design lanes' report files;
  - a handoff note in the project's state directory when a session ends mid-drive;
  - the `designer`'s artifact directory (`design.md`, `tokens.json`, `evidence/prove.md`, `evidence/contrast.mjs`).

## Contributing

Issues and pull requests are welcome.

- **Open an issue for a failure you hit:** the agent, what it did, and what it should have done. Most rules here started
  as one such failure.
- **A pull request changes one contract, or one rule across contracts.** Say which failure it prevents. A rule with no
  failure behind it is hard to judge.
- **Keep each contract self-contained.** No references to files outside `agents/` other than the project files listed
  above. No personal paths, credentials, or details specific to one product or vendor.
- **Keep the frontmatter valid:** `name`, `description`, `model`, `effort`, `tools`. The `name` must match the file
  name.

## License

[MIT](LICENSE). Copyright (c) 2026 the contributors.
