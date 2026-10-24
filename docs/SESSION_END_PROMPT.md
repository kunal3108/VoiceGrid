# VoiceGrid Session End Prompt

This file contains prompt templates for ending an AI assistant session on the VoiceGrid project. Copy the appropriate template and paste it as the final message of your session.

---

## Full Session End Prompt

Use this at the end of every session to ensure proper documentation and handoff.

---

```
We are ending this VoiceGrid session. Please complete the following end-of-session protocol:

1. SUMMARIZE COMPLETED WORK
   - List only work that is fully done and verified (not "in progress" or "planned")
   - For each item: what was implemented, what tests were run, what the result was
   - If something was started but not finished, describe its current state clearly

2. LIST FILES CREATED OR MODIFIED
   - Every file that was created or modified this session
   - One-line description of what changed in each file

3. LIST TESTS EXECUTED AND RESULTS
   - Every test that was actually run this session
   - The actual result (pass/fail, not "should pass")
   - If no tests were run, say so explicitly

4. UPDATE PROJECT_STATUS.md
   - Update "Last updated" to today's date
   - Update "Completed Work" with all verified work from this session
   - Update "In Progress" to reflect current state
   - Update "Blockers" — add new ones, remove resolved ones
   - Update "Latest Verified Tests" with actual test results
   - Update "Next Three Actions" with the specific next tasks

5. UPDATE SESSION_HANDOFF.md
   - Update "Date" and "AI assistant used"
   - Update "Work Completed" section
   - Update "Files Created or Modified"
   - Update "Tests Executed and Results"
   - Update "Concepts Learned" with any topics studied this session
   - Update "Current Problem" — the most important unresolved issue
   - Update "Next Immediate Action" — the single most important next step
   - Update "Latest Commit" with the actual git commit hash (if a commit was made)

6. UPDATE LEARNING JOURNALS
   - For every learning topic covered this session:
     - Change status: Not Started → In Progress → Completed
     - Update confidence level (honestly)
     - Update "Last Practiced" date
     - Add implementation reference if code was written

7. UPDATE GITHUB ISSUES (if remote repository is configured)
   - Comment on the active GitHub Issue with a progress summary
   - Close any issues where all acceptance criteria are now met
   - Create new issues for the next tasks if they don't exist

8. RECORD ACTUAL HOURS (if provided)
   - I will tell you how many hours this session took
   - Update PROJECT_STATUS.md "Actual hours completed" field

Please also output a session summary in this format:

---
SESSION SUMMARY — [date]

WORK COMPLETED (VERIFIED):
- [item 1]
- [item 2]

FILES MODIFIED:
- [file]: [what changed]

TESTS RUN:
- [test]: [result]

BLOCKERS:
- [blocker or "None"]

NEXT IMMEDIATE ACTION:
[single, specific, actionable task]
---
```

---

## Quick Session End Prompt

Use this for shorter sessions or quick updates.

---

```
End the VoiceGrid session. Update PROJECT_STATUS.md and SESSION_HANDOFF.md with:
- What was completed (verified only)
- Files modified
- Tests run and results
- Next immediate action

Output a brief session summary.
```

---

## Session End Without File Write Access

If the AI assistant cannot write files directly (e.g., ChatGPT in chat mode), use this prompt and apply the updates manually.

---

```
We are ending this VoiceGrid session. Since you cannot write files directly, please generate the proposed updates as text so I can apply them manually.

For each file below, generate the COMPLETE UPDATED CONTENT of the relevant sections (not just the diff):

1. PROJECT_STATUS.md — generate full updated content for:
   - "Last updated" line
   - "Completed Work" section
   - "In Progress" section
   - "Blockers" section
   - "Latest Verified Tests" section
   - "Next Three Actions" section

2. SESSION_HANDOFF.md — generate full updated content for:
   - All sections (this file is short; regenerate the whole thing)

3. Any learning journal entries that changed — generate the updated rows for the progress tracker table

Format each update as:
--- UPDATE FOR: [filename] ---
[updated content]
--- END UPDATE ---

I will copy each block and paste it into the appropriate file.
```

---

## What to Do If the Session Ended Without Following the Protocol

If a session ended abruptly without proper documentation, start the next session with:

---

```
The previous VoiceGrid session ended without completing the end-of-session protocol. 

Read SESSION_HANDOFF.md and PROJECT_STATUS.md to understand what was done.

Then do the following cleanup before we start new work:
1. Identify what was completed in the previous session (check git log if available)
2. Update SESSION_HANDOFF.md to reflect the actual state
3. Update PROJECT_STATUS.md to reflect the actual state
4. Confirm the next immediate action

Then tell me what you found and confirm the current state before we proceed.
```

---

## Tips for Session End

1. **Never skip the end-of-session protocol** — even a 5-minute session should update SESSION_HANDOFF.md
2. **Verify tests before updating status** — "Completed Work" means working and tested, not just implemented
3. **Be honest about what's unfinished** — "In Progress" with accurate state is more useful than false "Complete"
4. **The Next Immediate Action should be atomic** — specific enough to start without any ambiguity
5. **Commit before ending** — end every session with a commit so work is not lost
6. **Record actual hours** — even approximate hours help track the project's true pace against the 550h estimate
