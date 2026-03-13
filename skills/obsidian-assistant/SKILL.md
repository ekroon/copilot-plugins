---
name: obsidian-assistant
description: Use the Obsidian desktop CLI for vault-wide queries, app-native workflows, and actions that go beyond the Obsidian MCP tools. Prefer read-only commands by default, use MCP for structured note edits when it is the safer fit, confirm destructive actions before executing them, and fall back cleanly when either the MCP server or CLI is unavailable.
---

# Obsidian Assistant

Use this skill when the user wants to work with an Obsidian vault through the `obsidian` command-line interface, especially for actions that are broader or more app-native than the Obsidian MCP tools.

## Best-fit use cases

Reach for this skill when the user asks for any of the following:

- backlinks, outgoing links, unresolved links, or orphan/dead-end analysis
- vault search with context or counts
- task listing or task status updates through the CLI
- templates, daily note helpers, bookmarks, tabs, workspace, or random note workflows
- file history, diff, or restore operations
- plugin, theme, snippet, or command inspection and execution
- app-native operations such as opening notes, listing commands, or running a named Obsidian command
- generic Obsidian requests like "find orphan notes", "show backlinks", "search my vault", or "what tasks are in today's note" even if the user does not mention CLI or MCP

## When to prefer MCP instead

Prefer the Obsidian MCP tools when the job is mainly structured note automation, including:

- reading or batch-reading note contents
- writing or patching note content
- updating frontmatter
- managing tags in a direct, structured way
- moving notes/files with explicit confirmations

When both interfaces could solve the task, default to:

1. MCP for targeted note edits and metadata changes
2. CLI for richer vault analysis, UI/workspace actions, and Obsidian-native features not exposed by MCP

## Tool availability and fallback behavior

Handle tool availability explicitly instead of failing vaguely:

- If both MCP and CLI are available, choose the better interface for the job.
- If MCP is unavailable but the `obsidian` CLI exists, continue with CLI-only workflows and say that structured MCP operations are not currently available.
- If the CLI command is unavailable but MCP is available, switch to MCP for note reads, writes, frontmatter, tags, moves, and searches that MCP can cover.
- If the user asks for a CLI-only feature and the `obsidian` command is missing, say so clearly and offer the closest MCP-backed alternative if one exists.
- If the user asks for an MCP-style edit and the MCP server is missing, either perform the edit with the CLI if the command is safe and equivalent, or explain the limitation and ask whether to proceed with the CLI approach.
- If neither interface is available, stop and explain the blocker plainly rather than pretending the task can be completed.

Before relying on the CLI, verify that `obsidian` is installed and callable by running a lightweight check such as:

```bash
obsidian version
```

Before relying on MCP, verify that the needed Obsidian MCP tools are exposed in the current session. If that is uncertain, treat a failed lightweight MCP read call as MCP unavailability rather than continuing blindly.

If an interface fails at runtime after initially appearing available, surface the actual error, stop pretending the path is healthy, and either retry with the other interface or explain the blocker clearly.

## Operating rules

- Default to read-only commands first.
- Before any destructive action, explain what will happen and get user confirmation.
- Treat these as destructive or high-impact: `delete`, `history:restore`, `move`, `rename`, `property:remove`, `plugin:disable`, `plugin:uninstall`, `theme:set`, `theme:uninstall`, `snippet:disable`, and all `command id=...` calls.
- Prefer exact `path=` when the target note is known; use `file=` only when wiki-link style resolution is appropriate.
- If the user names a specific vault, pass `vault=<name>` explicitly.
- If the user does not name a vault, omit `vault=` and let the CLI use the active vault. If the command cannot proceed safely without a specific vault, ask the user.
- Keep commands transparent: show the planned `obsidian ...` command in the response before execution when risk is non-trivial.
- If a request mixes CLI-style analysis with structured edits, split the workflow: inspect with CLI, then edit with MCP.
- Do not assume the presence of both interfaces; check and adapt.
- For multi-note or bulk updates, confirm scope first: single note, folder, or whole vault. Summarize the affected scope before executing.
- The examples in this skill intentionally use Obsidian CLI `key=value` syntax because that is the command style this CLI accepts.

## Recommended workflow

1. Clarify the vault, file, path, or scope only if it is genuinely ambiguous.
2. Confirm which interface is available if availability is uncertain.
3. Start with the narrowest read-only command that answers the question.
4. Parse structured command output internally and present a concise human summary unless the user asks for raw output.
5. If the user wants a follow-up change, choose the safer interface:
   - MCP for note content/frontmatter/tag edits
   - CLI for app-native actions and CLI-only features
6. If the task naturally splits into "analyze then edit", do the analysis with CLI first and prefer MCP for the edit step.
7. Validate results after mutating commands when possible.

## Command patterns

### Read-only inspection

```bash
obsidian backlinks path="Projects/My Note.md" counts format=json
obsidian search:context query="agent memory" path="20-Resources" format=json
obsidian tasks path="Daily/2026-03-09.md" format=json
obsidian history path="Projects/My Note.md"
obsidian plugins format=json versions
obsidian commands filter="workspace"
```

### App-native actions

```bash
obsidian open path="Inbox/Capture.md"
obsidian daily:append content="- Follow up on MCP vs CLI comparison"
obsidian command id="workspace:split-right"
obsidian template:read name="Meeting Note" resolve title="Design Sync"
```

### Mixed CLI + MCP workflow

Example: find orphan notes with the CLI, then add a frontmatter flag with MCP only after the user confirms.

1. Use CLI:

```bash
obsidian orphans format=json
```

2. For any notes the user chooses to mark, switch to MCP for structured frontmatter updates.

### Fallback examples

If the CLI is missing:

- say that the `obsidian` command is unavailable
- continue with MCP if the user only needs note reads, writes, tags, frontmatter, or note moves

If MCP is missing:

- say that MCP-backed structured note operations are unavailable
- continue with the CLI for searches, backlinks, tasks, history, plugins, workspace actions, or narrowly targeted file edits supported by the CLI

If both are missing:

- stop and tell the user what is missing
- suggest installing/upgrading Obsidian for the CLI or enabling the MCP server for structured automation

## Trigger phrases

This skill is a strong match for prompts like:

- "Use the Obsidian CLI to find backlinks for this note"
- "Show backlinks for this Obsidian note"
- "What notes have no incoming links?"
- "Find orphan notes in my vault"
- "What unresolved links do I have in my vault?"
- "List my Obsidian commands, plugins, or themes"
- "Show tasks from today's daily note"
- "What tasks are in today's note?"
- "Search my vault with context for this phrase"
- "Search my Obsidian vault for this"
- "Diff this note against an older version"
- "Open this note in Obsidian" 
- "Use Obsidian CLI instead of MCP for this"
- "If MCP is down, can you still do this with the CLI?"
- "The obsidian command is missing; what can you still do?"

## Safety checklist

Before executing a write command, verify:

- the target vault and path are correct
- the command is as narrow as possible
- the scope is confirmed for bulk or multi-note operations
- the user has confirmed if the command can delete, restore, rename, move, or change app configuration
- there is a sensible post-action validation step

## Output style

- Keep replies short and practical.
- For read-only requests, summarize the result in plain language.
- For JSON-producing commands, parse the JSON internally and summarize it unless the user explicitly asks for the raw output.
- For risky operations, show the exact command and ask for confirmation.
- Mention when MCP would be safer for the edit portion of a workflow.
