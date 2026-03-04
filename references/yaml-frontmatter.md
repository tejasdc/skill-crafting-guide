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

- Restricts which tools the skill can access

```yaml
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"
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
license: MIT
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"
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

Injects dynamic context from shell commands or files when the skill loads. Runs at skill-load time, not at authoring time.

```yaml
context:
  - type: shell
    command: "git branch --show-current"
  - type: file
    path: "./package.json"
```

### agent

Specifies a subagent configuration for the skill. When set, the skill runs as an autonomous agent with its own tool access and context.

```yaml
agent:
  tools: ["Read", "Edit", "Bash"]
  model: sonnet
```

### model

Overrides the default model for this skill's execution. Use when a skill needs a specific capability level.

```yaml
model: opus
```

### argument-hint

Provides placeholder text shown to users when they type `/skill-name`. Helps users understand what arguments the skill expects.

```yaml
argument-hint: "<url-or-description>"
```

### hooks

Defines lifecycle hooks that run shell commands before/after the skill executes.

```yaml
hooks:
  pre: "echo 'Starting skill...'"
  post: "echo 'Skill complete.'"
```

### $ARGUMENTS substitution

When users invoke `/skill-name some text`, the `$ARGUMENTS` placeholder in your skill body is replaced with `some text`. This enables parameterized skills.

```markdown
# In SKILL.md body:
Analyze the following: $ARGUMENTS
```

### !command (dynamic context injection)

Lines starting with `!` execute shell commands at load time and inject their output into the skill body. Useful for including dynamic project state.

```markdown
# In SKILL.md body:
Current branch: !git branch --show-current
Recent commits: !git log --oneline -5
```

### user-invocable

Boolean. When `false`, the skill cannot be invoked by users via `/skill-name`. It becomes background knowledge that only Claude can use internally. Default is `true`.

```yaml
user-invocable: false
```

**Interaction with `disable-model-invocation`:**

| `user-invocable` | `disable-model-invocation` | Who can invoke |
|---|---|---|
| true (default) | false (default) | Both user and Claude |
| true | true | User only (via `/skill-name`) |
| false | false | Claude only (background knowledge) |
| false | true | Nobody (broken config -- avoid) |

---

## Why Frontmatter Matters

The frontmatter is the first level of progressive disclosure. It is always loaded in Claude's system prompt. Its purpose is to provide just enough information for Claude to know when each skill should be used without loading all of it into context.

A bad description means your skill either never loads (too vague) or loads too often (too broad). This is the single highest-leverage thing to get right.
