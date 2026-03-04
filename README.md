# skill-crafting-guide

A comprehensive skill for teaching Claude how to write high-quality skills — built by synthesizing multiple expert sources through a multi-agent research process.

## What It Does

When you ask Claude to create a skill, this skill loads and walks it through the complete process: defining use cases, structuring folders, writing frontmatter, crafting descriptions, applying progressive disclosure, choosing patterns, validating quality, testing, and distributing.

## How It Was Built

This skill was created using a **4-agent parallel research methodology** with cross-reviews and synthesis:

1. **Agent 1 — PDF Guide Analysis**: Extracted and synthesized Anthropic's official 33-page skill-writing PDF guide into actionable instructions
2. **Agent 2 — Existing Skills Analysis**: Reverse-engineered patterns from production skills (`superpowers:writing-skills`, `compound-engineering:create-agent-skills`, `skill-creator`) to find what actually works in practice
3. **Agent 3 — Web Research**: Gathered community best practices, forum discussions, and real-world skill-writing patterns from across the Claude ecosystem
4. **Agent 4 — PDF-as-Skill Conversion**: Treated the PDF guide itself as a skill-writing exercise, producing an alternative structure optimized for Claude's consumption

Each agent produced independent output. A **cross-review phase** had each agent evaluate the others' work, identifying gaps and contradictions. A final **synthesis step** merged the strongest elements from all four into a single unified skill.

### What Makes It Better Than Any Single Source

- **The PDF guide** is comprehensive but verbose (~33 pages). This skill distills it to ~300 lines with references.
- **Existing skills** have good patterns but lack the "why" behind decisions. This skill includes the reasoning.
- **Web research** surfaced real-world gotchas (name/folder mismatch is the #1 breaking error) that official docs underemphasize.
- **The synthesis** resolves contradictions between sources (e.g., when the PDF and existing skills disagree on structure).

## Installation

### Claude Code

Symlink into your skills directory:

```bash
ln -s /path/to/skill-crafting-guide ~/.claude/skills/skill-crafting-guide
```

### Codex

```bash
ln -s /path/to/skill-crafting-guide ~/.codex/skills/skill-crafting-guide
```

### Claude.ai

Upload the `SKILL.md` file via **Settings > Capabilities > Skills**.

## Skill Structure

```
skill-crafting-guide/
  SKILL.md                              # Main skill file (~300 lines)
  references/
    description-examples.md             # Good/bad description examples
    distribution-guide.md               # Distribution channels and methods
    patterns.md                         # 6 proven skill patterns with examples
    quick-checklist.md                  # Quality validation checklist
    testing-guide.md                    # Testing methodology
    troubleshooting.md                  # Common issues and fixes
    yaml-frontmatter.md                # Complete frontmatter reference
```

## Usage

Once installed, the skill triggers automatically when you ask Claude to:

- "Create a skill"
- "Write a SKILL.md"
- "Build a new skill"
- "Convert this into a skill"
- "Skill best practices"
- "Make a skill from this workflow"
- "Help me package this as a skill"
- "How do I structure a skill"

## License

MIT
