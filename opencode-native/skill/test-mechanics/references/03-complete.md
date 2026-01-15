# Step 3: Complete

**Tests:** State tracking, final output

## Instructions

1. **Confirm you loaded this file** - Tell the user: "I successfully loaded step 3."

2. **Finalize output** - Update output file:
   - Set frontmatter:
     ```yaml
     stepsCompleted: [1, 2, 3]
     lastStep: 3
     status: complete
     ```
   - Add a "Completion" section confirming all tests passed

3. **Report results** - Tell the user:
   - ✓ Skill loading worked (SKILL.md was loaded)
   - ✓ Reference loading worked (3 step files loaded on-demand)
   - ✓ Subagent delegation worked (bmad-pm--market-researcher responded)
   - ✓ State tracking worked (frontmatter updated each step)
   - ✓ Output generation worked (file created and updated)

4. **Show final output** - Display the contents of test-output.md

## Success Criteria

- All 3 steps completed in sequence
- Output file has complete frontmatter
- All mechanics validated
