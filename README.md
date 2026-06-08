# pi-brainstorm

> Fork of [@paulmupeters/pi-brainstorm](https://github.com/paulmupeters/pi-brainstorm) with `ask_user_question` tool integration.

A pi extension that adds a read-only `/brainstorm` mode with optional `ask_user_question` tool support.

## What it does

When brainstorm mode is active:
- allows only the `read` tool
- **also allows `ask_user_question` if the package `@juicesharp/rpiv-ask-user-question` is installed**
- blocks shell commands and file edits/writes
- keeps the conversation exploratory
- avoids unsolicited "you should do X next" suggestions
- gives a clear recommendation when you ask for the best option
- shows a visible reminder in the UI
- drafts a decision-oriented markdown brief when you finish
- can replace the brainstorm transcript with the reviewed brief in LLM context when you finish without saving or when you save and choose the context-preserving option

## Key difference from upstream

The original extension blocks all non-read tools during brainstorm mode. This fork:
- dynamically detects whether `ask_user_question` (`@juicesharp/rpiv-ask-user-question`) is registered
- if found, allows it alongside `read` so the agent can ask structured multi-choice questions to clarify ambiguous requests
- if not found, behaves exactly like the original (strictly read-only)
- no configuration needed — detection is automatic at runtime

## UX

- `/brainstorm` starts brainstorm mode
- `/brainstorm` again opens a small menu:
  - Continue brainstorming
  - Finish and summarize
  - Cancel and discard
- `/brainstorm finish` finishes directly
- `/brainstorm cancel` exits immediately without a summary
- `/brainstorm-summary-model` configures an optional dedicated summary model
- `Ctrl+Alt+B` is a shortcut for the same flow

While active, the footer/widget reminds you how to finish or cancel.

## Install / test

### Quick test

```bash
pi --no-extensions -e /path/to/brainstorm.ts
```

### Use from your normal pi setup

Install from npm:

```bash
pi install npm:@aalalice233/pi-brainstorm
```

Or copy `extensions/brainstorm.ts` into `~/.pi/agent/extensions/`.

### Optional: enable ask_user_question

If you also want to use structured questions during brainstorm:

```bash
pi install npm:@juicesharp/rpiv-ask-user-question
```

The integration is automatic — no config needed.

## Brief export

When you finish a brainstorm, the extension:
1. collects the conversation since brainstorm mode started
2. asks the current model, or an optional dedicated summary-model override, to draft a concise decision brief
3. opens that brief in an editor so you can tweak it
4. then offers:
   - `Brief to context`
   - `Brief to markdown`
   - `Brief to markdown and context`
   - `Continue brainstorming`
   - `Exit`

The generated brief uses `# Decision Brief: <topic>` when the session reached a clear decision, recommendation, or strong preference. If no firm conclusion emerged, it uses `# Brainstorm Brief: <topic>` and calls out the strongest current leaning without inventing certainty. It leads with the recommendation/current leaning, then covers rationale, alternatives, risks/open questions, and a transcript summary capped at 5 sentences.

Default save path:

```text
brainstorms/YYYY-MM-DD-topic.md
```

### Optional summary model override

By default, brainstorm summaries use the currently active Pi model.

You can persist a separate user-level summary model with:

```text
/brainstorm-summary-model
```

Or set it directly:

```text
/brainstorm-summary-model google/gemini-2.5-flash
/brainstorm-summary-model clear
```

The preference is stored globally in `~/.pi/agent/settings.json` under `piBrainstorm.summaryModel` and falls back to the active model when unset or unavailable.

If you choose **Brief to context** or **Brief to markdown and context**, the brainstorm transcript stays in session history, but future LLM context uses the reviewed brief instead of the full brainstorm exchange.

## Notes

- During brainstorm mode, the `read` tool is always enabled; `ask_user_question` is also enabled when the package is installed.
- The extension restores your previously active tools after finishing/canceling.
- If model-based brief generation is unavailable, the extension falls back to a simple markdown transcript.
- Tool detection happens at runtime — no config or restart needed when installing/removing `@juicesharp/rpiv-ask-user-question`.
