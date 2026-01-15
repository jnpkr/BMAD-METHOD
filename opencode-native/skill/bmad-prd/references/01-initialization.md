# Step 1: Initialization

**Progress: Step 1 of 11** | Next: Project Discovery

## Goal

Initialize the PRD workflow by detecting continuation state, discovering input documents, and setting up the document structure.

## Sequence

### 1. Check for Existing PRD

Look for an existing PRD file:
- `planning-artifacts/prd.md`
- `docs/prd.md`
- Ask user if they have a PRD elsewhere

**If found with incomplete steps**: Offer to continue from where it left off
**If found and complete**: Confirm if user wants to revise or start fresh
**If not found**: Proceed with fresh setup

### 2. Discover Input Documents

Search for and load context documents:

| Document Type | Search Pattern | Purpose |
|--------------|----------------|---------|
| Product Brief | `*brief*.md` | Vision and initial requirements |
| Research | `*research*.md` | Market/user research findings |
| Project Context | `**/project-context.md` | Existing project information |
| Project Docs | `docs/**/*.md` | Technical documentation |

Also check for sharded documents (folders with `index.md`).

### 3. Determine Project Type

Based on what you find:
- **Greenfield**: No existing codebase documentation
- **Brownfield**: Has existing project/architecture documentation

### 4. Report to User

Present your findings:

"I've discovered the following for your PRD workspace:

**Input Documents Found:**
- Product briefs: [count] files
- Research: [count] files
- Project docs: [count] files

**Project Type:** [Greenfield/Brownfield]

[If Brownfield]: This is an existing project. I'll focus on understanding what new features or changes you want to add.

Do you have any other documents to include, or shall we continue?"

### 5. Create Initial PRD

If starting fresh:
1. Create PRD file using template structure from @assets/prd-template.md
2. Initialize frontmatter with workflow state
3. Record input documents in frontmatter

### 6. Proceed

Once user confirms, proceed to Step 2: Project Discovery.

Update frontmatter: `stepsCompleted: [1]`

## Success Criteria

- Existing workflow state properly detected
- Input documents discovered and loaded
- Project type (greenfield/brownfield) identified
- User informed and confirmed
- PRD document initialized with proper structure
