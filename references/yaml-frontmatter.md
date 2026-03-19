# YAML Frontmatter Reference

The YAML frontmatter is how Claude decides whether to load your skill. It appears in Claude's system prompt, so it must be concise and accurate.

## Minimal Required Format

```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

That is all you need to start.

## Required Fields

### name (required)

- kebab-case only
- No spaces or capitals
- **MUST match the folder name exactly** (this is the most common breaking error)

```yaml
# Correct
name: my-cool-skill

# Wrong
name: My Cool Skill
name: my_cool_skill
name: MyCoolSkill
```

### description (required)

- MUST include BOTH: what the skill does AND when to use it (trigger conditions)
- Under 1024 characters
- No XML angle brackets (less-than or greater-than signs)
- Include specific tasks users might say
- Mention file types if relevant

**Structure:**

```
[What it does] + [When to use it] + [Key capabilities]
```

## Optional Fields

### disable-model-invocation

- Boolean. When `true`, prevents Claude from auto-triggering this skill. Users must invoke it manually via `/skill-name`.
- **Use when:** The skill has side effects (deploys code, submits to app stores, sends messages), contains real credentials, or should only run on explicit user request.
- Note: This is different from `user-invocable`. Setting `user-invocable: false` means only Claude can invoke the skill (background knowledge). Setting `disable-model-invocation: true` means only the user can invoke it (manual commands).

```yaml
disable-model-invocation: true
```

### license

- Use if making skill open source
- Common values: MIT, Apache-2.0

```yaml
license: MIT
```

### compatibility

- 1-500 characters
- Indicates environment requirements: intended product, required system packages, network access needs

```yaml
compatibility: Requires Node.js 18+. Works with Claude Code and Claude.ai.
```

### allowed-tools

- Tools Claude can use without asking permission when this skill is active.
- Comma-separated list of tool names.

```yaml
allowed-tools: Read, Grep, Glob
# Or with Bash restrictions:
allowed-tools: Bash(python *), Bash(npm *), WebFetch
```

### metadata

- Any custom key-value pairs
- Suggested: author, version, mcp-server

```yaml
metadata:
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

## Complete Example with All Fields

```yaml
---
name: skill-name
description: [required description]
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash(python *)
model: sonnet
context: fork
agent: Explore
argument-hint: "[topic]"
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
license: MIT
metadata:
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
---
```

**Note:** You would not normally set both `disable-model-invocation: true` and `context: fork` on the same skill. This example shows all fields for reference.

**Note on descriptions:** Use single-line strings for the `description` field. The skill indexer does not reliably parse YAML multiline indicators (`>-`, `|`). If your description is long, it can wrap naturally in the YAML -- just keep it on one logical line.

## Security Notes

**Allowed:**
- Any standard YAML types (strings, numbers, booleans, lists, objects)
- Custom metadata fields
- Long descriptions (up to 1024 characters)

**Forbidden:**
- XML angle brackets (less-than or greater-than signs) - security restriction because frontmatter appears in system prompt
- Code execution in YAML (uses safe YAML parsing)
- Skills named with "claude" or "anthropic" prefix (reserved)

## Advanced Fields

These fields unlock deeper Claude Code integration. Use them when your skill needs specific execution behavior beyond basic triggering.

### context

Set to `fork` to run the skill in an isolated subagent context. The skill content becomes the prompt that drives the subagent. The subagent gets a fresh context window with CLAUDE.md + your skill — it does NOT have access to conversation history.

```yaml
context: fork
```

**When to use:** Research tasks, verbose operations, tasks that benefit from a specific agent type's tool restrictions. See `agent` field below.

**When NOT to use:** Skills that provide guidelines/conventions without an actionable task (the subagent gets no task to execute). Skills that need conversation history or user interaction.

### agent

Specifies which subagent type executes the skill when `context: fork` is set. This is a simple string — NOT an object.

Options include built-in agents (`Explore`, `Plan`, `general-purpose`) or any custom subagent defined in `.claude/agents/`. If omitted, defaults to `general-purpose`.

```yaml
# Run research in a read-only Explore agent (uses Haiku, fast)
context: fork
agent: Explore

# Run in a Plan agent (inherits model, read-only)
context: fork
agent: Plan

# Run with full tool access (default if agent omitted)
context: fork
agent: general-purpose

# Use a custom subagent you defined in .claude/agents/
context: fork
agent: my-custom-reviewer
```

**How it works:** Skills with `context: fork` and subagents with a `skills` field are two sides of the same coin:

| Approach | System prompt | Task | Also loads |
|----------|--------------|------|------------|
| Skill with `context: fork` | From agent type (Explore, Plan, etc.) | SKILL.md content | CLAUDE.md |
| Subagent with `skills` field | Subagent's markdown body | Claude's delegation message | Preloaded skills + CLAUDE.md |

### model

Overrides the default model for this skill's execution. Use when a skill needs a specific capability level.

```yaml
model: opus
```

Valid values: `sonnet`, `opus`, `haiku`, or a full model ID (e.g., `claude-opus-4-6`).

### argument-hint

Provides placeholder text shown to users during autocomplete when they type `/skill-name`. Helps users understand what arguments the skill expects.

```yaml
argument-hint: "[issue-number]"
# or
argument-hint: "[filename] [format]"
```

### hooks

Defines lifecycle hooks scoped to this skill. Uses the same format as hooks in settings.json — NOT simple pre/post strings. See [Hooks in skills and agents](https://code.claude.com/docs/en/hooks#hooks-in-skills-and-agents) for the full configuration format.

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

### user-invocable

Boolean. When `false`, the skill cannot be invoked by users via `/skill-name`. It becomes background knowledge that only Claude can use internally. Default is `true`.

```yaml
user-invocable: false
```

**Interaction with `disable-model-invocation`:**

| `user-invocable` | `disable-model-invocation` | Who can invoke | When loaded into context |
|---|---|---|---|
| true (default) | false (default) | Both user and Claude | Description always in context, full skill loads when invoked |
| true | true | User only (via `/skill-name`) | Description not in context, full skill loads when you invoke |
| false | false | Claude only (background knowledge) | Description always in context, full skill loads when invoked |
| false | true | Nobody (broken config — avoid) | Not loaded |

---

## String Substitutions

Skills support variable substitutions in the markdown body (after the `---` closing frontmatter).

### $ARGUMENTS

When users invoke `/skill-name some text`, the `$ARGUMENTS` placeholder in your skill body is replaced with `some text`. If `$ARGUMENTS` is not present in the content, arguments are appended as `ARGUMENTS: <value>`.

```markdown
# In SKILL.md body:
Analyze the following: $ARGUMENTS
```

### $ARGUMENTS[N] and $N (positional access)

Access individual arguments by 0-based index. `$N` is shorthand for `$ARGUMENTS[N]`.

```markdown
# These are equivalent:
Migrate $ARGUMENTS[0] from $ARGUMENTS[1] to $ARGUMENTS[2].
Migrate $0 from $1 to $2.
```

Invoking `/migrate-component SearchBar React Vue` replaces `$0` → `SearchBar`, `$1` → `React`, `$2` → `Vue`.

### ${CLAUDE_SESSION_ID}

The current session ID. Useful for logging, creating session-specific files, or correlating skill output with sessions.

```markdown
Log the following to logs/${CLAUDE_SESSION_ID}.log:
$ARGUMENTS
```

### ${CLAUDE_SKILL_DIR}

The directory containing the skill's `SKILL.md` file. For plugin skills, this is the skill's subdirectory within the plugin, not the plugin root. Use this in shell commands to reference scripts or files bundled with the skill, regardless of the current working directory.

```markdown
Run validation: !`python ${CLAUDE_SKILL_DIR}/scripts/validate.py`
```

---

## Dynamic Context Injection: !`command`

The `` !`command` `` syntax runs shell commands **before** the skill content is sent to Claude. The command output replaces the placeholder inline — Claude only sees the result.

**Correct syntax** — backtick-wrapped:

```markdown
# In SKILL.md body:
Current branch: !`git branch --show-current`
Recent commits: !`git log --oneline -5`
PR diff: !`gh pr diff`
```

This is preprocessing, not something Claude executes. Each `` !`command` `` runs immediately when the skill loads, output replaces the placeholder, and Claude receives the fully-rendered prompt.

**Using ${CLAUDE_SKILL_DIR} for bundled scripts:**

```markdown
!`python ${CLAUDE_SKILL_DIR}/scripts/gather-context.py`
```

**Complete example combining dynamic context with subagent execution:**

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

---

## Extended Thinking

To enable extended thinking (thinking mode) in a skill, include the word **"ultrathink"** anywhere in your skill content. Claude will use extended thinking for deeper reasoning when processing the skill.

---

## Skill Character Budget

Skill descriptions are loaded into Claude's context so it knows what's available, but full skill content only loads when invoked. The description budget scales at **2% of the context window** with a fallback of 16,000 characters. If you have many skills, some may be excluded. Run `/context` to check for warnings. Override with `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable.

---

## Permissions

Three ways to control which skills Claude can invoke:

**Disable all skills:** Add `Skill` to deny rules in `/permissions`.

**Allow or deny specific skills:**
```
Skill(commit)       # Allow exact match
Skill(review-pr *)  # Allow prefix match with any arguments
Skill(deploy *)     # Deny prefix match (if in deny rules)
```

**Hide individual skills:** Add `disable-model-invocation: true` to their frontmatter.

---

## Open Standard

Claude Code skills follow the [Agent Skills](https://agentskills.io) open standard, which works across multiple AI tools. Claude Code extends the standard with `context: fork` (subagent execution), `agent` field, and `` !`command` `` (dynamic context injection).

---

## Why Frontmatter Matters

The frontmatter is the first level of progressive disclosure. It is always loaded in Claude's system prompt. Its purpose is to provide just enough information for Claude to know when each skill should be used without loading all of it into context.

A bad description means your skill either never loads (too vague) or loads too often (too broad). This is the single highest-leverage thing to get right.
