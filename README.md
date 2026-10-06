# grill-me

[![License: MIT](https://img.shields.io/badge/license-MIT-2563eb?style=flat-square)](LICENSE)
[![Codex](https://img.shields.io/badge/Codex-skill-111827?style=flat-square)](#codex)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-d97757?style=flat-square)](#claude-code)

A planning skill for Codex and Claude Code. It inspects the project and asks about decisions that would be costly to guess before implementation.

This fork adapts [Matt Pocock's grill-me / grilling workflow](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) into one portable [SKILL.md](SKILL.md).

[Install](#install) · [Usage](#usage) · [Workflow](#workflow) · [Differences](#differences-from-upstream) · [Credits](#credits)

## Differences from upstream

Compared with Matt Pocock's original [grill-me entry point](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) and [grilling procedure](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md):

| Area | Original | This fork |
| --- | --- | --- |
| Packaging | `grill-me` delegates to a separate `grilling` skill. | The complete procedure and Codex adaptations live in one portable `SKILL.md`. |
| Activation | The `grill-me` entry point requires user invocation. | Requests automatic use before implementation; explicit invocation remains available. |
| Questions | Numbered Markdown rounds with recommended answers. | Interactive choices through `request_user_input` in Codex or `AskUserQuestion` in Claude Code, with consequences for each option. |
| Context | Looks up environmental facts as questions require them. | Starts with reconnaissance and presents a sourced assumptions ledger before the questions. |
| Decision scope | Walks the decision tree until every branch is visited. | Puts consequential decisions in the question UI and proposes cosmetic choices for the user to veto. |
| Plan | Does not require a plan artifact. | Drafts and revises a session plan with scope, decisions, and verification steps; Codex keeps it in the conversation when no scratchpad is named. |
| Between rounds | Recomputes the decision frontier from the answers. | Also summarizes what changed, distinguishes verified facts from assumptions, and turns remaining risks into questions. |
| Ending early | Defines completion as an empty decision frontier. | Also supports accept-all and stop phrases, recording unresolved assumptions before proceeding. |
| Handoff | Waits for the user to confirm shared understanding before acting. | Starts work once decisions are settled, within the authorized scope. Normal permissions still apply to irreversible or external actions. |

The decision tree, dependency-aware rounds, recommended answers, and rule that the agent finds facts itself come from the original. This repository is a GitHub fork of [mattpocock/skills](https://github.com/mattpocock/skills), with the original history preserved.

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
