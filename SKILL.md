---
name: grill-me
description: "Interrogate the user's request before acting: investigate first, expose assumptions, and settle load-bearing decisions before any task that changes files, runs commands with effects, or performs multi-step work. Invoke automatically for every such request; it never needs to be named explicitly. Also use when the user asks to grill, challenge, clarify, or stress-test a question or plan. Skip only reading, explanations, status checks, one-line lookups, and decisions already settled in the conversation."
---

# Grill me

This skill contains the complete procedure. Automatic activation is the
normal mode; explicit invocation is optional.

## Codex adaptations

When running in Codex, apply these adaptations to the procedure below.
They take precedence where tool names, limits, or scratchpad handling differ.

- Use the session scratchpad directory named by the active system prompt for
  `plan.md`. If none is named, keep the plan in the conversation instead of
  creating a repository file.
- Use `request_user_input` for every decision round. It is the Codex equivalent
  of Claude's `AskUserQuestion`: choices belong in that interactive UI, never in
  a prose list or a normal chat question.
- After recon, every grillable task must have at least one interactive decision
  round before work starts. Do not declare the frontier empty merely because
  the recommended assumptions look reasonable. The only exceptions are when
  the user already settled every load-bearing decision in the conversation, or
  used the stop phrases described by the canonical skill.
- If `request_user_input` is unavailable in the current collaboration mode, do
  not emulate it with plain text and do not start the work. Pause and state that
  interactive grilling requires Plan mode, where `request_user_input` is
  available. Resume the recon/round workflow there.
- Treat the canonical `AskUserQuestion` limit as the limit exposed by the
  current Codex UI; never invent an unavailable tool.
- Announce activation in commentary, including whether it caused recon, a
  question round, or a pause.

## Procedure

Adapted from Matt Pocock's `grilling` primitive (github.com/mattpocock/skills,
MIT). The assumptions ledger, the load-bearing/cosmetic tiering and the
accept-all escape hatch are from Chase AI's `crucible` (github.com/chaseai-yt,
MIT); the between-rounds summary is from Jekudy's `grillme`; the
explore-before-you-rank rule is from Alex Reardon's `cook-me`.

Departures from all of them: the questions are asked with **AskUserQuestion**
rather than printed as markdown, the session is anchored to a **plan.md**, and
it ends by doing the work rather than by asking permission to.

## The shape

Map the work as a **design tree**: every decision branches into the decisions
that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose
prerequisites are already settled — the questions answerable *now*, without
guessing at answers not yet heard. Ask the whole frontier, wait, then recompute
it: settled decisions push the frontier outward and unblock what depended on
them. A question whose answer hinges on another question still open in this
round belongs to a **later** round, not this one.

Done when the frontier is empty. Not when the questions run out — when every
branch has been visited and nothing load-bearing is left silently assumed.

## The order

1. **Draft `plan.md`** in the session scratchpad directory named in the system
   prompt. Write it as the task is understood right now, guesses included and
   marked as guesses. This is the target the grilling shoots at; without it the
   interview drifts into an interesting conversation about nothing.
2. **Phase 0 — recon**, then the **rounds**. Both below.
3. **Rewrite `plan.md`** from the answers, and say in two or three lines what
   changed — a plan that comes back identical means the round asked nothing that
   mattered, and that is worth noticing out loud.
4. **Start the work.** No confirmation gate: the grilling was the confirmation,
   and asking «поехали?» after it is the ceremony this skill exists to remove.
   The only pauses that survive are the ones that were never about the plan —
   irreversible or outward-facing actions (merging, deploying, deleting,
   anything that leaves the machine), which follow their own standing rules.

`plan.md`, kept short — a page, not a document:

```markdown
# <задача одной строкой>

## Что просили
<дословно, как было сформулировано>

## Что меняется
<файлы и что в них — конкретно>

## Чего не трогаю
<границы: то, что рядом и остаётся как есть>

## Как проверю
<команда, тест, снимок — то, что даёт ответ, а не то, что «должно быть ок»>

## Решено в интервью
<решение — выбранный вариант — почему; одна строка на каждое>

## Допущения и открытые вопросы
<то, что осталось неразрешённым, и как я это трактую>
```

## Phase 0 — the assumptions ledger

Before asking anything, go and find out. Read the code, query the indexes, run
the cheap command. Then put **one list** in the chat of everything resolved
alone, with a source on every line:

```markdown
## Реестр допущений
_Поправьте одним ответом. Непрокомментированное считаю подтверждённым._
1. <допущение> — источник: <путь / строка индекса / вывод команды>
2. ...
```

The user corrects in one reply; silence confirms. Corrections that open real
questions become decisions in the rounds below.

This is the phase that pays for the skill. Every line in the ledger is a
question not asked. Skip the phase only when there is nothing to recon — a
greenfield idea with no repository and no files — and say that you skipped it.

## The rounds

Each round is one **AskUserQuestion** call. The tool caps a call at **four**
questions, so a wide frontier takes two calls back to back — that is fine, and
it is still one round.

### Two tiers, and the test that separates them

**Load-bearing** — a wrong answer costs a migration, a rewrite, a security hole,
or the user's trust. These go in the popup, one per question slot, and each
carries a line naming **what breaks if we guess wrong**.

**Cosmetic** — renameable, refactorable, swappable later. These never enter the
popup. They go out as one plain-text list, each with a recommendation and a
one-line rationale, vetoed by exception: silence is acceptance.

The test is the «what breaks» line itself. If it comes out vague — «было бы
аккуратнее», «потом сложнее читать» — the decision is cosmetic. Demote it and
give the slot back to a question that earns it.

### Explore before you rank

A question that ranks options assumes the goal is already known. When it is not,
the round's job is to surface intent, not to offer a menu — and a ranking whose
right answer depends on an unresolved question higher in the tree belongs to the
next round, not to this one's options. Do not rush to rank because the user
handed you a list.

### Each question

- **2–4 options**, concrete and mutually exclusive. Not «да/нет» when the real
  fork is between three approaches.
- **The recommendation first**, labelled `(Рекомендую)`. Having no
  recommendation means the question is not yet understood well enough to ask.
- **`description` states the consequence**, not the option restated. What
  becomes true, what it costs, what it forecloses.
- `header` ≤ 12 characters; option labels 1–5 words.
- Free-form answers arrive through the automatic **Other** — never add an
  «Other» option manually.
- Reach for `preview` when the options are things to *look at*: layouts, message
  formats, competing snippets, directory shapes.

### Between rounds

When the round moved something — a correction, an unexpected pick, a new branch
— print a short summary before the next one:

- **Что понял** — the facts now settled.
- **Допущения** — each marked **проверено** or **предположение**. The
  distinction is the point; an unmarked list hides which half is guessing.
- **Риски → вопросы** — each risk turned into a specific question for the next
  round. A risk that produces no question was not a risk.

When every answer matched the recommendation, do not print the summary. Print
one line — «возражений не было» — and treat it as a signal, not a result.

### The escape hatch

The user can say **«принимаю все рекомендации»** at any point: every open
decision locks at its recommended answer, logged as such in the plan, and the
work starts. Say this out loud when the load-bearing tier runs past six
questions. It stays a phrase, never an option in the popup and never a question
of its own — a hatch offered every round is an invitation to give up early.

## Facts are my job; decisions are the user's

Never ask what can be looked up. A question answerable by `grep`, by reading a
file, by an index or by running a command is not a question for the user — find
the answer first and put the *decision* it exposes to them instead.

Look things up in parallel with the round when they are not prerequisites: a
running lookup is an unsettled prerequisite only for the questions downstream of
it. Ask the rest of the frontier now.

## When to stop, and how

- **The frontier empties.** The normal ending.
- **The user says stop** — «хватит», «поехали», «делай». Stop that instant. Move
  every unresolved branch into *Допущения* in `plan.md`, stated as the
  interpretation being acted on, and start.
- **The question is ungrillable.** Some things cannot be settled by talking —
  how an interaction should feel, whether one long form beats three pages. They
  need something to react to. Say so, build the throwaway version, look at it,
  and come back with the answer in one line. Talking through an ungrillable
  question is how a session balloons.
- **The scope is too large.** Two hundred questions is not thoroughness, it is
  the wrong unit of work. Propose splitting first, then grill one piece.

## What a good session looks like

The user disagrees with something. A round where every answer was the
recommendation proved nothing — say so plainly. Later rounds visibly build on
earlier answers. At the end the user could defend each choice to someone who was
not there.

The failure mode is not too few questions. It is a long, agreeable session
producing a plan the user nodded at and does not own.

## Do not grill

Reading, explaining, status, one-line lookups, and anything already decided in
this conversation. The rule is «before any task that changes or runs
something» — a question is not a task. Re-grilling a decision the user just made
is not diligence, it is not listening.
