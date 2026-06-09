# pi-brainstorm

> Fork of [@paulmupeters/pi-brainstorm](https://github.com/paulmupeters/pi-brainstorm) with expanded read-only tool support.

A pi extension that adds a read-only `/brainstorm` mode with dynamic tool detection. During brainstorm, the agent can use `read`, `find`, and any of the following optional tools if registered in the environment:
- `grep` — for pattern search within files
- `ask_user_question` — for structured clarifying questions
- `subagent` / `get_subagent_result` — for deep codebase exploration

## What it does

When brainstorm mode is active:
- allows `read`, `find` and any detected optional tools (`grep`, `ask_user_question`, `subagent`, `get_subagent_result`)
- blocks shell commands and file edits/writes
- keeps the conversation exploratory
- avoids unsolicited "you should do X next" suggestions
- gives a clear recommendation when you ask for the best option
- shows a visible reminder in the UI
- drafts a decision-oriented markdown brief when you finish
- can replace the brainstorm transcript with the reviewed brief in LLM context when you finish without saving or when you save and choose the context-preserving option

## Key difference from upstream

The original extension blocks all non-read tools during brainstorm mode. This fork:
- dynamically detects which tools are available in the current pi environment (`grep`, `ask_user_question`, `subagent`, `get_subagent_result`)
- allows all detected tools alongside `read` and `find`, giving the agent richer capabilities during brainstorm (e.g. searching code, asking structured questions, exploring codebases in background)
- silently omits tools that are not registered — no configuration needed
- shows a dynamic allowed-tool list in the system prompt so the agent knows exactly what it can use

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

### Optional: enable additional brainstorm tools

The extension detects these optional tools at runtime — install any of the packages below and the tool is automatically available in brainstorm mode:

| Tool | Package |
|---|---|
| `grep` | Built-in to pi, no extra install needed |
| `ask_user_question` | `@juicesharp/rpiv-ask-user-question` |
| `subagent` / `get_subagent_result` | Built-in to pi, no extra install needed |

For example:

```bash
pi install npm:@juicesharp/rpiv-ask-user-question
```

All tool detection is automatic — no config needed.

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

- During brainstorm mode, `read` and `find` are always enabled; `grep`, `ask_user_question`, `subagent`, and `get_subagent_result` are enabled when their respective packages/tools are registered.
- The extension restores your previously active tools after finishing/canceling.
- If model-based brief generation is unavailable, the extension falls back to a simple markdown transcript.
- Tool detection happens at runtime — no config or restart needed when installing/removing packages or when tools become available/unavailable.
