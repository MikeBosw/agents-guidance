---
name: manage-pr
description: Manages full PR lifecycle — creates feature branch, commits, opens PR, waits for feedback, handles review comments inline
user-invocable: true
---

# manage-pr

Orchestrates the full PR lifecycle. Follow each step in order. Skip steps that have already been handled. Sometimes you will be going through all the steps of creating a whole new PR, and sometimes you will merely be responding to a bit of new feedback for an existing PR. See steps below for more detail.

## Step 1: Ensure a feature branch

Run `git branch --show-current` to check the current branch.

- If on `master` or `main`: create a `feature/<topic>` branch, deriving the topic name from the staged/unstaged changes. Switch to it.
- If already on a `feature/*` branch: continue on it (skip to step 2)

## Step 2: Commit changes

If changes are already committed, skip to step 3.

Follow the commit protocol defined in CLAUDE.md:

1. `git add` the changed files (prefer adding specific files by name, not `git add -A`).
2. Run `pre-commit run` and fix any issues it reports. Re-stage and retry until hooks pass.
3. Commit with a message that follows CLAUDE.md rules (lowercase first letter unless acronym/proper noun, concise, no trailing period) and include the `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>` trailer.

## Step 3: Push and create PR (if needed)

Check whether a PR already exists for this branch:

```
gh pr view --json number 2>/dev/null
```

- If a PR exists: note the PR number and skip to Step 5.
- If no PR exists:
  1. Push with `git push -u origin HEAD`.
  2. Create the PR with `gh pr create` following CLAUDE.md's PR protocol: use a `feature/` branch, include a `## Summary` with 1-3 bullet points and a `## Test plan` checklist, and append the Claude Code generation footer.
  3. Note the PR number and continue to Step 4.

## Step 4: Wait for feedback (new PRs only)

This step applies only when a PR was just created in Step 3. Skip it if the PR already existed.

Tell the user you are waiting 7 minutes for Copilot and human reviewers to leave comments, then run:

```
sleep 420
```

After waking, proceed to Step 5.

## Step 5: Handle feedback

Determine the repo owner/name from `gh repo view --json nameWithOwner -q .nameWithOwner`.

Fetch both types of comments:

```
gh api repos/{owner}/{repo}/pulls/{number}/comments
gh api repos/{owner}/{repo}/issues/{number}/comments
```

For each comment that has not already been addressed (i.e., has no `[claude]` reply to the last copilot/human message(s)):

1. Read the comment carefully. Evaluate it against CLAUDE.md rules and your own engineering judgment.
2. Copilot feedback is highly useful and highly fallible — accept good suggestions, reject bad ones with a brief rationale.
3. If the feedback warrants a code change: make the change, stage, and note it for the final commit.
4. Reply **inline at the comment-thread level** (not at the PR level) using the GitHub API. For pull request review comments, reply in the thread:
   ```
   gh api repos/{owner}/{repo}/pulls/{number}/comments \
     -X POST \
     -f body="[claude] <your reply>" \
     -F in_reply_to=COMMENT_ID
   ```
   For issue-level comments, reply on the issue:
   ```
   gh api repos/{owner}/{repo}/issues/{number}/comments \
     -X POST \
     -f body="[claude] <your reply>"
   ```

After handling all comments: run `pre-commit run`, fix any issues, commit the fixes (with the same commit conventions from Step 2), and push.

## Step 6: thinking about oversights implied by the issues found (if any)

For all bug fixes implemented in reaction to this feedback, please consider what KINDS of oversights led those bugs to be introduced. Enumerate them specifically.

## Step 7: hunting for potential lurking issues due to the oversights identified

This part is tricky. What other bugs, minor landmines, inconsistencies, broken assumptions, readability issues, unnecessary complexity, typing laziness, or other issues might sneakily have been introduced by the kinds of oversights identified?

Look for them in the code. Confirm or refute your own suspicions and report to the user on any new bugs uncovered by this analysis.

If operating in non-interactive mode, please:
1. Post a comment to the PR with your findings, including what your analysis process was and what you looked for
2. Attempt to fix issues you've surfaced, including pre-existing bugs, and push those fixes to the PR
3. Follow up with a comment on the PR saying which of the issues, if any, you addressed
