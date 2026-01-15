# Step 1: Setup

**Tests:** Reference file loading, output file creation

## Instructions

1. **Confirm you loaded this file** - Tell the user: "I successfully loaded step 1 from the skill's references folder."

2. **Create output file** - Create `test-output.md` using the template from `assets/output-template.md`

3. **Update state** - Set frontmatter in output file:
   ```yaml
   stepsCompleted: [1]
   lastStep: 1
   ```

4. **Report to user** - Show what you created and ask if ready for step 2.

## Success Criteria

- This file was loaded from skill's base directory
- Output file created with template structure
- Frontmatter has stepsCompleted: [1]
