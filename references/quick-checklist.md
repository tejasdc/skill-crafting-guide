# Quick Checklist

Use this checklist to validate your skill before and after upload. Items are grouped by severity -- fix all CRITICAL items first, then HIGH, then MEDIUM, then LOW.

## Severity Guide

| Severity | Meaning | Impact |
|----------|---------|--------|
| CRITICAL | Breaks skill | Upload fails, skill never loads, or silent corruption |
| HIGH | Degrades quality | Skill loads but undertriggers, instructions ignored, or poor UX |
| MEDIUM | Optimization | Skill works but misses opportunities for better performance |
| LOW | Nice-to-have | Polish items that improve discoverability or maintainability |

## Before You Start

- [ ] Identified 2-3 concrete use cases
- [ ] Tools identified (built-in or MCP)
- [ ] Reviewed the skill-writing-guide and example skills
- [ ] Planned folder structure
- [ ] Decided: generic skill or project-specific skill?

## During Development

### CRITICAL (breaks skill if violated)

- [ ] `name` field MUST match folder name exactly
- [ ] SKILL.md file exists (exact spelling, case-sensitive)
- [ ] YAML frontmatter has `---` delimiters (opening and closing)
- [ ] No XML angle brackets (less-than or greater-than signs) anywhere in frontmatter
- [ ] `name` field: kebab-case, no spaces, no capitals

### HIGH (degrades quality significantly)

- [ ] `description` includes WHAT it does and WHEN to use it, with trigger phrases users would actually say
- [ ] Instructions are clear and actionable (specific commands, not vague guidance)
- [ ] Error handling included (common issues with specific fixes)
- [ ] SKILL.md uses progressive disclosure (under 500 lines, detailed content in references/)
- [ ] No README.md inside skill folder
- [ ] Description uses single-line string (not YAML multiline indicators)

### MEDIUM (optimization)

- [ ] Examples provided (at least one common scenario)
- [ ] References clearly linked with content descriptions (what each file contains)
- [ ] Negative triggers included ("Do NOT use for X -- use Y instead")
- [ ] `disable-model-invocation: true` for side-effect workflows (deploy, submit, send)
- [ ] Reference files over 100 lines have a table of contents
- [ ] No time-sensitive information in skill content

### LOW (nice-to-have)

- [ ] Gerund naming convention (`processing-pdfs` over `pdf-processor`)
- [ ] `license` field for open-source distribution
- [ ] `compatibility` field for environment requirements
- [ ] `metadata` field (author, version, category, tags)
- [ ] Reference files split by domain with descriptive filenames

## Before Upload

- [ ] Tested triggering on obvious tasks (exact trigger phrases)
- [ ] Tested triggering on paraphrased requests (varied wording)
- [ ] Verified does NOT trigger on unrelated topics
- [ ] Functional tests pass (correct outputs for test inputs)
- [ ] Tool integration works (if applicable -- MCP calls succeed)
- [ ] Compressed as .zip file (for upload to Claude.ai)

## After Upload

- [ ] Test in real conversations
- [ ] Monitor for under/over-triggering
- [ ] Collect user feedback
- [ ] Iterate on description and instructions

## Success Metrics

**Quantitative:**
- Skill triggers on 90% of relevant queries
- Completes workflow in X tool calls (compare with/without skill)
- 0 failed API calls per workflow

**Qualitative:**
- Users do not need to prompt Claude about next steps
- Workflows complete without user correction
- Consistent results across sessions
- New user can accomplish the task on first try with minimal guidance
