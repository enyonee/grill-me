# grill-me

[![License: MIT](https://img.shields.io/badge/license-MIT-2563eb?style=flat-square)](LICENSE)
[![Codex](https://img.shields.io/badge/Codex-skill-111827?style=flat-square)](#codex)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-d97757?style=flat-square)](#claude-code)

A planning skill for Codex and Claude Code. It inspects the project and asks about decisions that would be costly to guess before implementation.

This fork adapts [Matt Pocock's grill-me / grilling workflow](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) into one portable [SKILL.md](SKILL.md).

[Install](#install) · [Usage](#usage) · [Workflow](#workflow) · [Differences](#differences-from-upstream) · [Credits](#credits)

## Install

Choose the agent you use. The commands below install the skill for your user account and require Git.

### Codex

```bash
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/enyonee/grill-me.git ~/.agents/skills/grill-me
```

The path follows the [official Codex skill documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills). If the skill does not appear, restart Codex.

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone --depth 1 https://github.com/enyonee/grill-me.git ~/.claude/skills/grill-me
```

This is the personal skill location documented by [Claude Code](https://code.claude.com/docs/en/skills#choose-where-skills-load).

You can also install manually: place [SKILL.md](SKILL.md) in a folder named `grill-me` inside your agent's skill directory. The file includes the complete procedure and its license notice.

## Usage

In **Codex**, mention the skill in your prompt:

```text
$grill-me Add team invitations to this app. Inspect the existing auth flow first.
```

In **Claude Code**, invoke it as a slash command:

```text
/grill-me Add team invitations to this app. Inspect the existing auth flow first.
```

The skill also asks the agent to activate it automatically before implementation tasks. It skips reading, explanations, status checks, and decisions already settled in the conversation.

Use an interactive session with the agent's question UI. In Codex, the skill pauses if `request_user_input` is unavailable; follow its instruction to switch to Plan mode and retry.

### Example question

An illustrative question after inspecting an invitation flow:

> Who can invite a teammate?
>
> **Workspace admins (recommended):** keep membership changes under the existing admin role.
>
> **Any member:** allow everyone to invite teammates; the permission model must support this.
>
> **Workspace owners:** reserve invitations for owners; admins cannot invite people.

The actual question appears in the agent's interactive question UI. The next round depends on your answer.

### Ending the interview

| Say | Effect |
| --- | --- |
| `принимаю все рекомендации` (accept all recommendations) | Accept the recommended answers to open decisions and start work. |
| `делай` (do it), `поехали` (let's go), or `хватит` (enough) | Stop questioning, record the remaining assumptions, and proceed. |

Normal permission requirements still apply to irreversible or external actions.

## Workflow

1. **Draft the plan.** Record the request, affected files, scope, checks, and unresolved assumptions. In Codex, use the session scratchpad when available; otherwise keep the plan in the conversation.
2. **Inspect the context.** Read the relevant files and find facts the agent can establish without asking you. Present an assumptions ledger with sources.
3. **Resolve consequential decisions.** Ask through the host's question UI, with a recommended answer and the consequences of each option. Defer questions until their prerequisites are settled. Keep cosmetic choices out of the interview.
4. **Update the plan.** Record the answers and remaining assumptions. Explain what changed.
5. **Start the work.** Once the decisions are resolved, proceed within the authorized scope without a redundant final confirmation.

## Differences from upstream

| Area | This adaptation |
| --- | --- |
| Packaging | The full procedure lives in one `SKILL.md`; no companion `grilling` skill is required. |
| Activation | Requests automatic use before implementation, as well as explicit invocation. |
| Questions | Uses `request_user_input` in Codex and `AskUserQuestion` in Claude Code, with each host's supported limits. |
| Assumptions | Adds a sourced ledger and separates consequential decisions from cosmetic choices. |
| Plan | Keeps decisions and verification steps in a session plan. |
| Handoff | Starts work after the interview, within the authorized scope. |

The repository is a GitHub fork of [mattpocock/skills](https://github.com/mattpocock/skills). The default branch contains this focused adaptation; the original history is preserved.

## Update

Run the command for your installation:

```bash
# Codex
git -C ~/.agents/skills/grill-me pull --ff-only

# Claude Code
git -C ~/.claude/skills/grill-me pull --ff-only
```

## Credits

- [Matt Pocock](https://github.com/mattpocock/skills): the core `grilling` procedure and `grill-me` entry point.
- [Chase AI](https://github.com/chaseai-yt): the `crucible` influences credited in the skill, including the assumptions ledger and decision tiers.
- [Jekudy](https://github.com/Jekudy/grillme-skill): the between-round summaries credited in the skill.
- Alex Reardon: the `cook-me` influence credited in the skill, which calls for exploring intent before ranking options.

See [SKILL.md](SKILL.md) for the complete instructions and attribution.

## License

[MIT](LICENSE). The upstream copyright notice is preserved in both `LICENSE` and the portable `SKILL.md`.
