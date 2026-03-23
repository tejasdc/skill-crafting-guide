# Distribution and Sharing Guide

## Installing Skills with npx (Recommended)

The easiest way to install skills from GitHub is `npx skills add` — a CLI built by Vercel for the Agent Skills open standard. **Any GitHub repo containing a valid `SKILL.md` is automatically compatible.** No special configuration, manifest, or registry step is needed.

### Install commands

```bash
# From a GitHub repo (shorthand)
npx skills add yourorg/your-skill-repo

# From a full GitHub URL
npx skills add https://github.com/yourorg/your-skill-repo

# Install a specific skill from a multi-skill repo
npx skills add yourorg/your-skills --skill my-specific-skill

# Install globally (user-level, available across all projects)
npx skills add yourorg/your-skills --global

# List available skills in a repo without installing
npx skills add yourorg/your-skills --list
```

### How it works

1. Clones the repo to a temp directory
2. Scans for `SKILL.md` files in `skills/`, root, and other standard locations
3. Presents an interactive picker if multiple skills are found
4. Copies to `.agents/skills/<name>/` (canonical location)
5. Creates symlinks into agent-specific directories (e.g., `~/.claude/skills/<name>/` for Claude Code)

### What your repo needs

Nothing beyond a valid skill structure. If your repo has a `SKILL.md` with valid `name` and `description` frontmatter, it works. The CLI discovers skills automatically.

**Single-skill repo:**
```
your-skill/
  SKILL.md
  references/       # optional
  scripts/           # optional
```

**Multi-skill repo:**
```
your-skills-repo/
  skills/
    skill-one/
      SKILL.md
    skill-two/
      SKILL.md
  README.md
```

### Other install methods

**Claude.ai (manual upload):**
1. Download or zip the skill folder
2. Upload via Settings then Capabilities then Skills

**Claude Code (manual placement):**
Place the skill folder in `~/.claude/skills/` (global) or `.claude/skills/` (project-level).

**Organization-level:**
Admins can deploy skills workspace-wide with automatic updates and centralized management.

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

Use this template in your repo's README. Lead with the one-liner — most users will never need the manual steps.

```markdown
## Install

npx skills add yourorg/your-skill-repo

That's it. The skill is now available in Claude Code (and other compatible agents).

### Alternative installation

If you prefer manual installation:

1. Clone: `git clone https://github.com/yourorg/your-skill-repo`
2. Copy the skill folder to `~/.claude/skills/` (global) or `.claude/skills/` (project-level)

### Verify

Ask Claude: "Set up a new project in [Your Service]" — the skill should trigger automatically.
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
