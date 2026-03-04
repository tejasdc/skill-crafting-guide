# Skill Patterns

These patterns emerged from skills created by early adopters and internal teams. They represent common approaches that work well, not prescriptive templates.

## Table of Contents

- [Choosing Your Approach: Problem-First vs. Tool-First](#choosing-your-approach-problem-first-vs-tool-first)
- [Pattern 1: Sequential Workflow Orchestration](#pattern-1-sequential-workflow-orchestration)
- [Pattern 2: Multi-MCP Coordination](#pattern-2-multi-mcp-coordination)
- [Pattern 3: Iterative Refinement](#pattern-3-iterative-refinement)
- [Pattern 4: Context-Aware Tool Selection](#pattern-4-context-aware-tool-selection)
- [Pattern 5: Domain-Specific Intelligence](#pattern-5-domain-specific-intelligence)
- [Pattern 6: Router SKILL.md](#pattern-6-router-skillmd)
- [Combining Patterns](#combining-patterns)

## Choosing Your Approach: Problem-First vs. Tool-First

Think of it like Home Depot. You might walk in with a problem - "I need to fix a kitchen cabinet" - and an employee points you to the right tools. Or you might pick out a new drill and ask how to use it for your specific job.

Skills work the same way:

- **Problem-first:** "I need to set up a project workspace" -- Your skill orchestrates the right MCP calls in the right sequence. Users describe outcomes; the skill handles the tools.
- **Tool-first:** "I have Notion MCP connected" -- Your skill teaches Claude the optimal workflows and best practices. Users have access; the skill provides expertise.

Most skills lean one direction. Knowing which framing fits your use case helps you choose the right pattern below.

---

## Pattern 1: Sequential Workflow Orchestration

**Use when:** Your users need multi-step processes in a specific order.

**Example structure:**

```markdown
## Workflow: Onboard New Customer

### Step 1: Create Account
Call MCP tool: `create_customer`
Parameters: name, email, company

### Step 2: Setup Payment
Call MCP tool: `setup_payment_method`
Wait for: payment method verification

### Step 3: Create Subscription
Call MCP tool: `create_subscription`
Parameters: plan_id, customer_id (from Step 1)

### Step 4: Send Welcome Email
Call MCP tool: `send_email`
Template: welcome_email_template
```

**Key techniques:**
- Explicit step ordering
- Dependencies between steps (data from Step 1 used in Step 3)
- Validation at each stage
- Rollback instructions for failures

---

## Pattern 2: Multi-MCP Coordination

**Use when:** Workflows span multiple services.

**Example: Design-to-development handoff**

```markdown
### Phase 1: Design Export (Figma MCP)
1. Export design assets from Figma
2. Generate design specifications
3. Create asset manifest

### Phase 2: Asset Storage (Drive MCP)
1. Create project folder in Drive
2. Upload all assets
3. Generate shareable links

### Phase 3: Task Creation (Linear MCP)
1. Create development tasks
2. Attach asset links to tasks
3. Assign to engineering team

### Phase 4: Notification (Slack MCP)
1. Post handoff summary to #engineering
2. Include asset links and task references
```

**Key techniques:**
- Clear phase separation
- Data passing between MCPs (asset links from Phase 2 used in Phase 3)
- Validation before moving to next phase
- Centralized error handling

---

## Pattern 3: Iterative Refinement

**Use when:** Output quality improves with iteration.

**Example: Report generation**

```markdown
## Iterative Report Creation

### Initial Draft
1. Fetch data via MCP
2. Generate first draft report
3. Save to temporary file

### Quality Check
1. Run validation script: `scripts/check_report.py`
2. Identify issues:
   - Missing sections
   - Inconsistent formatting
   - Data validation errors

### Refinement Loop
1. Address each identified issue
2. Regenerate affected sections
3. Re-validate
4. Repeat until quality threshold met

### Finalization
1. Apply final formatting
2. Generate summary
3. Save final version
```

**Key techniques:**
- Explicit quality criteria
- Iterative improvement with clear stopping conditions
- Validation scripts (code is deterministic; language interpretation is not)
- Know when to stop iterating

---

## Pattern 4: Context-Aware Tool Selection

**Use when:** Same outcome, different tools depending on context.

**Example: File storage**

```markdown
## Smart File Storage

### Decision Tree
1. Check file type and size
2. Determine best storage location:
   - Large files (>10MB): Use cloud storage MCP
   - Collaborative docs: Use Notion/Docs MCP
   - Code files: Use GitHub MCP
   - Temporary files: Use local storage

### Execute Storage
Based on decision:
- Call appropriate MCP tool
- Apply service-specific metadata
- Generate access link

### Provide Context to User
Explain why that storage was chosen
```

**Key techniques:**
- Clear decision criteria
- Fallback options
- Transparency about choices (tell the user why a particular path was chosen)

---

## Pattern 5: Domain-Specific Intelligence

**Use when:** Your skill adds specialized knowledge beyond tool access.

**Example: Financial compliance**

```markdown
## Payment Processing with Compliance

### Before Processing (Compliance Check)
1. Fetch transaction details via MCP
2. Apply compliance rules:
   - Check sanctions lists
   - Verify jurisdiction allowances
   - Assess risk level
3. Document compliance decision

### Processing
IF compliance passed:
  - Call payment processing MCP tool
  - Apply appropriate fraud checks
  - Process transaction
ELSE:
  - Flag for review
  - Create compliance case

### Audit Trail
- Log all compliance checks
- Record processing decisions
- Generate audit report
```

**Key techniques:**
- Domain expertise embedded in logic (compliance rules, fraud checks)
- Compliance before action
- Comprehensive documentation and audit trails
- Clear governance (what happens when checks fail)

---

---

## Pattern 6: Router SKILL.md

**Use when:** Your skill is complex with many reference files and you want maximum token efficiency.

**The approach:** SKILL.md is a lean ~120-200 line overview/table of contents that routes to reference files. It contains:
1. A workflow overview table mapping each phase to its reference file
2. The top 5-7 critical gotchas (surfaced at level 2 so they are always available)
3. A reference index mapping files to their contents
4. The single most critical step inline (the "climax step" -- see below)

**Example structure:**

```markdown
# App Store Launch

## Workflow Overview

| Phase | Method | Reference |
|-------|--------|-----------|
| 1. Build | xcodebuild CLI | See references/build-and-upload.md for: archive, export, upload commands |
| 2. API Setup | App Store Connect API | See references/api-setup.md for: JWT auth, helper functions |
| 3. Metadata | API + Browser | See references/metadata-and-assets.md for: screenshots, age rating, pricing |
| 4. Submit | API | Inline below (climax step) |

## Critical Gotchas (Read Before Starting)
1. Content rights is under General > App Information, not where you expect
2. Privacy questionnaire resets if you change any answer
[... top 5-7 items ...]

## Submit for Review (Climax Step)
[The most critical code/procedure lives here, not in a reference file]

## Reference Index

| File | Contents |
|------|----------|
| build-and-upload.md | Archive commands, exportOptions.plist, upload via xcrun |
| api-setup.md | JWT generation, helper functions, app lookup |
| metadata-and-assets.md | Screenshots, age rating, pricing, localization |
| gotchas.md | Full troubleshooting reference (30+ items) |
```

**The "climax step" exception:** Every workflow has one step that is the whole point. For a deployment skill, it is the deploy command. For a submission skill, it is the submit call. Keep this code directly in SKILL.md even when everything else is in references. Claude needs the most important step without an extra file read.

**Key techniques:**
- Workflow overview table with linked references
- Critical gotchas surfaced at level 2 (always loaded)
- Descriptive reference links telling Claude WHAT each file contains
- Climax step kept inline
- Maximum token efficiency -- Claude loads only the reference files it needs

---

## Combining Patterns

Many real-world skills combine multiple patterns. A design handoff skill might use Pattern 2 (Multi-MCP Coordination) for the overall flow, Pattern 3 (Iterative Refinement) for the design review step, and Pattern 5 (Domain-Specific Intelligence) for accessibility compliance checks.

Choose your primary pattern based on the core workflow, then layer in secondary patterns where they add value.
