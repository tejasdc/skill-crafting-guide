# Advanced Features

Most skills only need frontmatter + instructions. These features unlock deeper integration when you need it.

## Dynamic Context Injection

The `!` followed by a backtick-wrapped command syntax runs shell commands **before** the skill content reaches Claude. The command output replaces the placeholder inline — Claude only sees the result, not the command.

Example skill using dynamic context injection:

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
---
```

In the body of that skill, lines like these inject live data:

```
- PR diff: !` gh pr diff `
- PR comments: !` gh pr view --comments `
- Changed files: !` gh pr diff --name-only `
```

(Note: the spaces inside the backticks above are intentional to prevent this reference file from being preprocessed. In actual usage, remove the spaces.)

When the skill runs, each command executes immediately, output replaces the placeholder, and Claude receives the fully-rendered prompt with actual data. This is preprocessing — not something Claude executes.

Use `${CLAUDE_SKILL_DIR}` to reference scripts bundled with your skill regardless of working directory:

```
python ${CLAUDE_SKILL_DIR}/scripts/gather-context.py
```

Prefix that line with the bang-backtick syntax in your actual skill.

**WARNING:** Do not put bang-backtick examples directly in SKILL.md — the preprocessor will try to execute them. Document this syntax only in reference files.

## Running Skills in a Subagent

Add `context: fork` to run a skill in an isolated subagent context. The skill content becomes the prompt that drives the subagent. The subagent gets a fresh context window with CLAUDE.md + your skill as the prompt — it does NOT have access to your conversation history.

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:
1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

The `agent` field specifies which subagent type executes the skill. Options include built-in agents (`Explore`, `Plan`, `general-purpose`) or any custom subagent defined in `.claude/agents/`. If omitted, defaults to `general-purpose`.

**When to use `context: fork`:**
- Tasks that produce verbose output you don't need in your main context
- Research tasks that can return a summary
- Tasks that benefit from a specific agent type's tool restrictions (e.g., `Explore` for read-only research)

**When NOT to use `context: fork`:**
- Skills that provide guidelines or conventions (no actionable task for the subagent)
- Skills that need conversation history or back-and-forth with the user

## String Substitutions

Skills support these variable substitutions in the markdown body:

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed when invoking the skill |
| `$ARGUMENTS[N]` | Access a specific argument by 0-based index (e.g., `$ARGUMENTS[0]`) |
| `$N` | Shorthand for `$ARGUMENTS[N]` (e.g., `$0`, `$1`, `$2`) |
| `${CLAUDE_SESSION_ID}` | Current session ID — useful for session-specific logging or file names |
| `${CLAUDE_SKILL_DIR}` | Directory containing the skill's SKILL.md — use in shell commands to reference bundled scripts |

If `$ARGUMENTS` is not present in the content, arguments are appended as `ARGUMENTS: <value>`.

**Example with positional arguments:**

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2. Preserve all existing behavior and tests.
```

Invoking `/migrate-component SearchBar React Vue` replaces `$0` with `SearchBar`, `$1` with `React`, `$2` with `Vue`.

## Extended Thinking

To enable extended thinking (thinking mode) in a skill, include the word **"ultrathink"** anywhere in your skill content. Claude will use extended thinking for more complex reasoning when processing the skill.

## Skill Character Budget

Skill descriptions are loaded into context so Claude knows what's available. If you have many skills, they may exceed the character budget. The budget scales dynamically at **2% of the context window**, with a fallback of 16,000 characters. Run `/context` to check for a warning about excluded skills. Override with the `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable.

## Open Standard

Claude Code skills follow the [Agent Skills](https://agentskills.io) open standard, which works across multiple AI tools. Claude Code extends the standard with `context: fork`, subagent execution, and dynamic context injection.
