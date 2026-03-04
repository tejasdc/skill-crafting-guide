---
name: skill-crafting-guide
description: Step-by-step guide for building high-quality skills for Claude. Use when user asks to "create a skill", "write a SKILL.md", "build a new skill", "convert this into a skill", "skill best practices", "make a skill from this workflow", "help me package this as a skill", or "how do I structure a skill". Covers frontmatter, descriptions, file structure, progressive disclosure, testing, and distribution.
---

# Skill Writing Guide

## Overview

A skill is a folder containing instructions that teach Claude how to handle specific tasks or workflows. Instead of re-explaining preferences every conversation, you teach Claude once and benefit every time.

This guide walks you through building a skill from scratch. Follow these steps in order. For deeper reference material, consult the files in `references/`.

## Critical Rules (Read First)

These are the most costly mistakes. Violating any of these will break your skill or cause silent failures:

1. **Name MUST match folder name.** If the folder is `my-cool-skill/`, the frontmatter must have `name: my-cool-skill`. A mismatch can cause upload failures or skill-loading errors.
2. **File MUST be named exactly `SKILL.md`** (case-sensitive). Not `skill.md`, not `Skill.md`.
3. **No XML angle brackets** (less-than or greater-than signs) anywhere in frontmatter. Security restriction; will cause rejection.
4. **No "claude" or "anthropic" in the name field.** Reserved prefixes.
5. **Description MUST include both WHAT and WHEN.** A description without trigger phrases means your skill never loads.
6. **Use single-line strings for descriptions.** The skill indexer does not reliably parse YAML multiline indicators (`>-`, `|`).
7. **Keep SKILL.md under 500 lines / 5,000 words.** Move detailed content to `references/`.

## Step 1: Define Your Use Cases

Before writing anything, identify 2-3 concrete use cases your skill should enable.

**Write them down in this format:**

```
Use Case: [Name]
Trigger: User says "[example phrase]" or "[example phrase]"
Steps:
1. [First action]
2. [Second action]
3. [Third action]
Result: [Expected outcome]
```

**Ask yourself:**
- What does a user want to accomplish?
- What multi-step workflows does this require?
- Which tools are needed (built-in or MCP)?
- What domain knowledge or best practices should be embedded?

**Decide: generic or project-specific?**
- **Generic skills** are reusable across projects. Use placeholders (`YOUR_API_KEY`, `YOUR_TEAM_ID`), write for broad audiences, and are suitable for public distribution.
- **Project-specific skills** contain real credentials, config values, or domain data for a particular project. They are kept internal, often use `disable-model-invocation: true`, and are valid when the skill is a reference card for a specific deployment or codebase.
- **When hardcoded values are okay:** Internal team skills, project-specific reference cards where the whole point is having the real values readily available.
- **When to genericize:** Public distribution, skills that could apply to multiple projects, anything shared outside your team.

**Three common skill categories:**
1. **Document and Asset Creation** - Consistent, high-quality output (designs, documents, code). Key techniques: embedded style guides, templates, quality checklists.
2. **Workflow Automation** - Multi-step processes with consistent methodology. Key techniques: step-by-step validation gates, templates, iterative refinement.
3. **MCP Enhancement** - Workflow guidance on top of MCP tool access. Key techniques: coordinating multiple MCP calls, domain expertise, error handling.

## Step 2: Create the Folder Structure

```
your-skill-name/           # kebab-case, no spaces/underscores/capitals
  SKILL.md                  # Required - main skill file (exact spelling)
  scripts/                  # Optional - executable code (Python, Bash, etc.)
  references/               # Optional - documentation loaded as needed
  assets/                   # Optional - templates, fonts, icons
```

**Critical naming rules:**
- **Folder name MUST match the `name` field in frontmatter.** This is the single most common breaking error. If the folder is `notion-project-setup/`, the frontmatter must say `name: notion-project-setup`.
- Folder name: kebab-case only. `notion-project-setup` is correct. `Notion Project Setup`, `notion_project_setup`, `NotionProjectSetup` are all wrong.
- **Recommended: use gerund (verb-ing) naming** when it reads naturally. `launching-ios-app` is preferred over `ios-app-launcher`. `processing-pdfs` over `pdf-processor`. This follows Anthropic's recommended convention. However, matching folder name to `name` field is MORE important than gerund convention -- never create a mismatch to achieve a gerund name.
- File name: Must be exactly `SKILL.md` (case-sensitive). Not `skill.md`, not `SKILL.MD`.
- No README.md inside the skill folder. All documentation goes in SKILL.md or references/.

## Step 3: Write the YAML Frontmatter

The frontmatter is how Claude decides whether to load your skill. This is the most important part to get right.

```yaml
---
name: your-skill-name
description: What it does and when to use it. Include trigger phrases.
---
```

**Required fields:**
- `name` - kebab-case, no spaces or capitals. **MUST match folder name exactly.**
- `description` - Must include WHAT the skill does AND WHEN to use it (trigger conditions). Under 1024 characters. Use a single-line string.

**The description formula:**

```
[What it does] + [When to use it] + [Key capabilities]
```

Write descriptions that include specific task phrases users would actually say. Mention relevant file types if applicable. See `references/description-examples.md` for good and bad examples.

**Optional fields:** `license`, `compatibility`, `allowed-tools`, `disable-model-invocation`, `metadata` (author, version, mcp-server, category, tags). See `references/yaml-frontmatter.md` for the complete reference.

**When to use `disable-model-invocation: true`:**
- Skills that modify external state (deploy, submit, publish, send)
- Skills containing real credentials or secrets
- Skills where accidental auto-triggering could cause harm
- Slash-command-style skills the user should explicitly invoke

When set, Claude cannot auto-trigger the skill. Users must invoke it manually via `/skill-name`.

**Security restrictions - NEVER include:**
- XML angle brackets (less-than or greater-than signs) in frontmatter
- "claude" or "anthropic" in the name field (reserved)

## Step 4: Write the Instructions Body

After the frontmatter closing `---`, write the actual instructions in Markdown.

**Recommended structure:**

```markdown
# Your Skill Name

## Instructions

### Step 1: [First Major Step]
Clear explanation of what happens.

### Step 2: [Next Step]
Include specific commands, parameters, or actions.

## Examples

### Example 1: [Common scenario]
User says: "[trigger phrase]"
Actions:
1. [Action]
2. [Action]
Result: [Outcome]

## Common Issues

### [Error type]
If you see "[error message]":
1. [Fix step]
2. [Fix step]
```

**Best practices for instructions:**
- Be specific and actionable. Write `Run python scripts/validate.py --input {filename}` not `Validate the data before proceeding.`
- Include error handling with specific conditions and fixes
- Reference bundled resources clearly: `Before writing queries, consult references/api-patterns.md for rate limiting guidance`
- Put critical instructions at the top, use `## Important` or `## Critical` headers
- Use bullet points and numbered lists, not walls of text

**Avoid ambiguous language:**

Bad: `Make sure to validate things properly`

Good: `CRITICAL: Before calling create_project, verify: project name is non-empty, at least one team member assigned, start date is not in the past`

## Step 5: Apply Progressive Disclosure

Skills use a three-level system to minimize token usage:

1. **First level (YAML frontmatter):** Always loaded in Claude's system prompt (~100 tokens). Provides just enough info for Claude to know WHEN to load the skill.
2. **Second level (SKILL.md body):** Loaded when Claude thinks the skill is relevant. Contains the full instructions.
3. **Third level (Linked files):** Additional files in the skill directory that Claude can choose to navigate and discover only as needed.

**The rule:** Keep SKILL.md focused on core instructions. Keep it under 500 lines / 5,000 words. Move detailed documentation, API references, extended examples, and lookup tables to `references/` and link to them from SKILL.md.

**Token economics:** Every token in a skill competes with conversation history. Do not explain things Claude already knows (what JWT is, how REST APIs work, basic programming concepts). Only add context Claude does not have: specific endpoints, non-obvious field types, gotchas from real experience.

**The "router SKILL.md" pattern (recommended):** Keep SKILL.md as a lean ~120-200 line overview that routes to reference files. Include a workflow overview table, the top 5-7 critical gotchas, and a reference index. All detailed code, API documentation, and extended procedures live in reference files. This minimizes the token footprint when the skill loads while still providing enough context to start working.

**The "climax step" exception:** If your skill has one critical step that is the whole point of the workflow (e.g., the final submission call, the deploy command), keep that code directly in SKILL.md even when everything else is in references. Claude needs the most important step without an extra file read.

**Surfacing critical info at level 2:** Always put the top 5-7 most important items (gotchas, warnings, breaking issues) directly in SKILL.md, even when a reference file has the full list. This prevents the most costly mistakes without requiring reference file loads.

### Reference File Organization

How you organize reference files matters for Claude's ability to find and load the right content.

**Split by DOMAIN, not by phase number.** Claude navigates by relevance, not sequence. When a user asks about screenshot errors, Claude should load `gotchas.md`, not `phase-6.md`.

**Recommended guidelines:**
- **Size:** 50-300 lines per reference file. Under 50 is too granular (merge it). Over 300 should be split further.
- **Table of contents:** Add one for any reference file over 100 lines so Claude can navigate efficiently.
- **Descriptive filenames:** `api-setup.md` not `phase-2.md`. `gotchas-and-errors.md` not `appendix-a.md`.
- **Descriptive links from SKILL.md:** Write "See references/api-setup.md for: JWT auth, helper functions, app lookup" rather than just linking to the file. This helps Claude decide whether to load the file without opening it.
- **One level deep:** SKILL.md links to reference files directly. Reference files should not chain to other reference files.

### Cross-Referencing Between Skills

When you have related skills (e.g., a generic workflow skill and a project-specific reference skill), use negative triggers to point users to the right one:

```yaml
# In the generic skill's description:
description: ... Do NOT use for Cortex-specific builds -- use app-store-submission-guide instead.

# In the project-specific skill's description:
description: ... Do NOT use for general iOS app publishing -- use ios-app-store-launch instead.
```

## Step 6: Choose Your Pattern

Most skills follow one of five proven patterns. Pick the one that fits your use case. See `references/patterns.md` for full examples of each.

| Pattern | Use When |
|---------|----------|
| Sequential Workflow | Multi-step processes in a specific order |
| Multi-MCP Coordination | Workflows spanning multiple services |
| Iterative Refinement | Output quality improves with iteration |
| Context-Aware Tool Selection | Same outcome, different tools depending on context |
| Domain-Specific Intelligence | Specialized knowledge beyond tool access |
| Router SKILL.md | Complex skills with many reference files |

**Problem-first vs. Tool-first framing:**
- Problem-first: "I need to set up a project workspace" - Your skill orchestrates the right tools in the right sequence. Users describe outcomes; the skill handles the tools.
- Tool-first: "I have Notion MCP connected" - Your skill teaches Claude optimal workflows. Users have access; the skill provides expertise.

## Step 7: Validate with the Quality Checklist

Before considering your skill complete, run through this checklist. See `references/quick-checklist.md` for the full version with severity ratings.

**CRITICAL (breaks skill if violated):**
- [ ] `name` field matches folder name exactly
- [ ] SKILL.md file exists (exact spelling, case-sensitive)
- [ ] YAML frontmatter has `---` delimiters (opening and closing)
- [ ] No XML angle brackets anywhere in frontmatter

**HIGH (degrades quality significantly):**
- [ ] `description` includes WHAT and WHEN with trigger phrases
- [ ] Instructions are clear and actionable (specific commands, not vague guidance)
- [ ] Error handling included (common issues with specific fixes)
- [ ] SKILL.md is under 500 lines; detailed content in references/

**MEDIUM (optimization):**
- [ ] Examples provided (at least one common scenario)
- [ ] References clearly linked with content descriptions
- [ ] Negative triggers to prevent over-triggering
- [ ] `disable-model-invocation: true` for side-effect workflows

**Triggering checks:**
- [ ] Tested triggering on obvious tasks
- [ ] Tested triggering on paraphrased requests
- [ ] Verified it does NOT trigger on unrelated topics

## Step 8: Test and Iterate

See `references/testing-guide.md` for the full testing methodology. The short version:

1. **Triggering tests** - Does the skill load at the right times? Test 10-20 queries. Track automatic vs. manual invocation.
2. **Functional tests** - Does it produce correct outputs? Check: valid outputs generated, API calls succeed, error handling works, edge cases covered.
3. **Performance comparison** - Compare the same task with and without the skill. Count tool calls and tokens consumed.

**Pro tip:** Iterate on a single task before expanding. Get one workflow working perfectly, then extract the winning approach into a skill.

**Iteration signals:**
- Undertriggering (skill does not load when it should): Add more detail and keywords to the description
- Overtriggering (skill loads for irrelevant queries): Add negative triggers, be more specific
- Instructions not followed: Keep instructions concise, put critical ones at top, move verbose content to references/

## Step 9: Distribute

See `references/distribution-guide.md` for full details.

**For individual use:** Place in Claude Code skills directory or upload via Claude.ai Settings at Capabilities then Skills.

**For teams:** Admins can deploy skills workspace-wide with automatic updates and centralized management.

**For public sharing:**
1. Host on GitHub with a clear README (separate from SKILL.md -- the README is for humans, SKILL.md is for Claude)
2. Document in your MCP repo if applicable
3. Include installation instructions, example usage, and screenshots

**For programmatic use:** The API provides `/v1/skills` endpoint and `container.skills` parameter for the Messages API.

## Common Issues

### Name/Folder Mismatch (Most Common Breaking Error)
Folder is `my-cool-skill/` but frontmatter says `name: cool-skill-helper`. Fix: make them identical. The name field MUST match the folder name exactly.

### Skill Will Not Upload
- **"Could not find SKILL.md"**: File not named exactly `SKILL.md` (case-sensitive). Verify with `ls -la`.
- **"Invalid frontmatter"**: Missing `---` delimiters, unclosed quotes, or other YAML formatting errors.
- **"Invalid skill name"**: Name has spaces or capitals. Use kebab-case only.

### Skill Does Not Trigger
Revise description field. Ask: Is it too generic? Does it include trigger phrases users would actually say? Does it mention relevant file types?

**Description debugging technique:** Ask Claude "When would you use the [skill name] skill?" Claude will quote the description back. Compare what it says with what you intended. Adjust based on the gap.

### Instructions Not Followed
Keep instructions concise. Use bullet points. Put critical instructions at the top. Move detailed reference to separate files. Consider bundling validation scripts for critical checks.

### MCP Connection Issues
1. Verify MCP server is connected (Claude.ai: Settings then Extensions)
2. Check authentication (API keys valid, permissions granted, OAuth tokens refreshed)
3. Test MCP independently without the skill
4. Verify tool names match MCP server documentation (case-sensitive)

For the complete troubleshooting reference, see `references/troubleshooting.md`.
