# Distribution and Sharing Guide

## Current Distribution Model

### How Individual Users Get Skills

1. Download the skill folder
2. Zip the folder (if needed)
3. Upload to Claude.ai via Settings then Capabilities then Skills
4. Or place in Claude Code skills directory

### Organization-Level Skills

- Admins can deploy skills workspace-wide (shipped December 18, 2025)
- Automatic updates when admin publishes new versions
- Centralized management

## An Open Standard

Skills are published as an open standard (Agent Skills). Like MCP, skills should be portable across tools and platforms. The same skill should work whether you are using Claude or other AI platforms. Some skills take advantage of platform-specific capabilities; authors can note this in the `compatibility` field.

## Using Skills via API

For programmatic use cases (building applications, agents, or automated workflows), the API provides direct control over skill management and execution.

**Key capabilities:**
- `/v1/skills` endpoint for listing and managing skills
- Add skills to Messages API requests via the `container.skills` parameter
- Version control and management through the Claude Console
- Works with the Claude Agent SDK for building custom agents

**Note:** Skills in the API require the Code Execution Tool beta, which provides the secure environment skills need to run.

### When to Use API vs. Claude.ai

| Use Case | Best Surface |
|----------|-------------|
| End users interacting with skills directly | Claude.ai / Claude Code |
| Manual testing and iteration during development | Claude.ai / Claude Code |
| Individual, ad-hoc workflows | Claude.ai / Claude Code |
| Applications using skills programmatically | API |
| Production deployments at scale | API |
| Automated pipelines and agent systems | API |

## Recommended Approach for Public Sharing

### 1. Host on GitHub

- Public repo for open-source skills
- Clear README with installation instructions
- Example usage and screenshots

**Important:** The README.md is for human visitors. It is separate from your skill folder, which should NOT contain a README.md. All Claude-facing documentation goes in SKILL.md or references/.

### 2. Document in Your MCP Repo

- Link to skills from MCP documentation
- Explain the value of using both together
- Provide quick-start guide

### 3. Create an Installation Guide

```markdown
## Installing the [Your Service] Skill

1. Download the skill:
   - Clone repo: `git clone https://github.com/yourcompany/skills`
   - Or download ZIP from Releases

2. Install in Claude:
   - Open Claude.ai, go to Settings then Skills
   - Click "Upload skill"
   - Select the skill folder (zipped)

3. Enable the skill:
   - Toggle on the [Your Service] skill
   - Ensure your MCP server is connected

4. Test:
   - Ask Claude: "Set up a new project in [Your Service]"
```

## Positioning Your Skill

How you describe your skill determines whether users understand its value and actually try it.

### Focus on Outcomes, Not Features

**Good (outcome-focused):**
> "The ProjectHub skill enables teams to set up complete project workspaces in seconds -- including pages, databases, and templates -- instead of spending 30 minutes on manual setup."

**Bad (feature-focused):**
> "The ProjectHub skill is a folder containing YAML frontmatter and Markdown instructions that calls our MCP server tools."

### Highlight the MCP + Skills Story

> "Our MCP server gives Claude access to your Linear projects. Our skills teach Claude your team's sprint planning workflow. Together, they enable AI-powered project management."

This framing helps users understand why they need both the MCP connector AND the skill.

## Official Resources

- **Public skills repository:** GitHub anthropics/skills - Contains Anthropic-created skills you can customize
- **Document Skills:** PDF, DOCX, PPTX, XLSX creation
- **Example Skills:** Various workflow patterns
- **Partner Skills Directory:** Asana, Atlassian, Canva, Figma, Sentry, Zapier, and more
