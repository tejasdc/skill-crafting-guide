# Description Field Examples

The description is the single most important field in your skill. It determines whether Claude loads your skill at the right time.

## The Formula

```
[What it does] + [When to use it] + [Key capabilities]
```

## Good Descriptions

### Specific and actionable

```yaml
description: Analyzes Figma design files and generates developer handoff
  documentation. Use when user uploads .fig files, asks for "design specs",
  "component documentation", or "design-to-code handoff".
```

Why it works: States the exact input (.fig files), the output (handoff docs), and multiple trigger phrases users would actually say.

### Includes trigger phrases

```yaml
description: Manages Linear project workflows including sprint planning,
  task creation, and status tracking. Use when user mentions "sprint",
  "Linear tasks", "project planning", or asks to "create tickets".
```

Why it works: Lists the specific words that should activate the skill. Claude can match on these even when the request is phrased differently.

### Clear value proposition

```yaml
description: End-to-end customer onboarding workflow for PayFlow. Handles
  account creation, payment setup, and subscription management. Use when
  user says "onboard new customer", "set up subscription", or "create
  PayFlow account".
```

Why it works: Describes the full workflow scope, names the product, and provides concrete trigger phrases.

### With negative triggers (to prevent over-triggering)

```yaml
description: Advanced data analysis for CSV files. Use for statistical
  modeling, regression, clustering. Do NOT use for simple data exploration
  (use data-viz skill instead).
```

Why it works: Explicitly states what the skill is NOT for, preventing it from loading when a more appropriate skill exists.

### With scope clarification

```yaml
description: PayFlow payment processing for e-commerce. Use specifically
  for online payment workflows, not for general financial queries.
```

Why it works: Draws a clear boundary around the skill's domain.

## Bad Descriptions

### Too vague

```yaml
# Bad
description: Helps with projects.
```

Problem: No trigger phrases, no specificity. Claude cannot determine when to load this.

### Missing triggers

```yaml
# Bad
description: Creates sophisticated multi-page documentation systems.
```

Problem: Describes what it does but not WHEN to use it. No user-facing trigger phrases. Users do not say "create sophisticated multi-page documentation systems."

### Too technical, no user triggers

```yaml
# Bad
description: Implements the Project entity model with hierarchical
  relationships.
```

Problem: Written for developers reading source code, not for matching against user requests. No one says "implement the Project entity model."

### Too broad

```yaml
# Bad
description: Processes documents
```

Problem: Would trigger on virtually any document-related request, causing over-triggering.

## Description Debugging

If your skill is not triggering correctly, ask Claude:

> "When would you use the [skill name] skill?"

Claude will quote the description back. Compare what Claude says with what you intended. Adjust based on the gap.

## Positioning for Humans (README, not SKILL.md)

When writing about your skill for humans (README, documentation, marketing), focus on outcomes not features:

**Good (outcome-focused):**
> "The ProjectHub skill enables teams to set up complete project workspaces in seconds -- including pages, databases, and templates -- instead of spending 30 minutes on manual setup."

**Bad (feature-focused):**
> "The ProjectHub skill is a folder containing YAML frontmatter and Markdown instructions that calls our MCP server tools."

**Highlighting the MCP + skills story:**
> "Our MCP server gives Claude access to your Linear projects. Our skills teach Claude your team's sprint planning workflow. Together, they enable AI-powered project management."
