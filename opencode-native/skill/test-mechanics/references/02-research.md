# Step 2: Research

**Tests:** Subagent delegation via task tool

## Instructions

1. **Confirm you loaded this file** - Tell the user: "I successfully loaded step 2."

2. **Delegate to subagent** - Use the `task` tool to delegate to the `bmad-pm--market-researcher` subagent:
   - subagent_type: "bmad-pm--market-researcher"
   - prompt: "Briefly describe what market research involves in 2-3 sentences. This is a test of subagent delegation."
   - description: "Test subagent delegation"

3. **Report result** - Show the subagent's response to the user.

4. **Update state** - Update output file frontmatter:
   ```yaml
   stepsCompleted: [1, 2]
   lastStep: 2
   ```

5. **Append to output** - Add a "Research" section to the output file with the subagent's response.

## Success Criteria

- Task tool successfully invoked market-researcher subagent
- Subagent returned a response
- Output file updated with research section
- Frontmatter updated to stepsCompleted: [1, 2]
