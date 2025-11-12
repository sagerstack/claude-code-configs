---
description: Git operations helper - checkout new branches, push changes, or merge from other branches with smart conflict handling
arguments:
  - name: action
    description: "Git action to perform: 'checkout', 'push', or 'merge'"
    required: true
  - name: branch-name
    description: "Name of the branch to create/checkout (mandatory if action=checkout) or merge from (mandatory if action=merge)"
    required: false
---

Execute git operations with intelligent workflow management.

**Arguments:**
1. **action** (required): Either "checkout", "push", or "merge"
2. **branch-name** (mandatory if action=checkout or action=merge): Name of branch to create/checkout or merge from

**Usage Examples:**
- `/git checkout feature/mvp2-sentiment-analysis` - Create and checkout new branch
- `/git push` - Add, commit, and push changes on current branch
- `/git merge main` - Merge latest main branch from remote into current branch

## Workflow Implementation

### Action: checkout

**Step 1: Check Current Branch Status**
1. Run `git status` to check for uncommitted changes
2. Identify if there are:
   - Untracked files
   - Modified files (not staged)
   - Staged files (not committed)

**Step 2: Handle Uncommitted Changes**
If there are ANY uncommitted files (untracked, modified, or staged):
1. Display the current status output to user
2. Ask user: "You have uncommitted changes on the current branch. Would you like to commit and push these changes before switching branches? (Y/N)"
3. Wait for user response
4. **If user responds "Y" or "Yes"** (case insensitive):
   - Run `git add .` to stage all changes
   - Generate a meaningful commit message based on the changed files (analyze git status to understand what was modified)
   - Run `git commit -m "<meaningful-message>"` following the commit message format from CLAUDE.md
   - Run `git push` to push changes to remote on current branch
   - Proceed to Step 3
5. **If user responds "N" or "No"** (case insensitive):
   - Display: "Operation cancelled. Please handle uncommitted changes manually before switching branches."
   - Exit without creating/checking out new branch

**Step 3: Create and Checkout New Branch**
If no uncommitted changes OR after successfully committing changes:
1. Run `git branch <branch-name>` to create new branch
2. Run `git checkout <branch-name>` to switch to new branch
3. Display: "Successfully created and checked out branch: <branch-name>"
4. Exit

**Error Handling:**
- If branch already exists: Display error and ask if user wants to checkout existing branch (don't create new one)
- If git operations fail: Display error message and exit

### Action: push

**Step 1: Stage All Changes**
1. Run `git add .` to stage all changes
2. Verify files were staged successfully

**Step 2: Generate Meaningful Commit Message**
1. Run `git status` to see what files changed
2. Run `git diff --staged --stat` to understand the nature of changes
3. Analyze the changes and generate a meaningful commit message following these guidelines:
   - Use conventional commit format: `<type>(<scope>): <description>`
   - Types: feat, fix, update, docs, refactor, test, chore
   - Include the "Generated with Claude Code" footer per CLAUDE.md
   - Keep message concise (1-2 sentences) focusing on "why" rather than "what"
   - Example: `feat(US-025): implement LunarCrush authentication setup with Bearer token`

**Step 3: Commit Changes**
1. Run `git commit -m "$(cat <<'EOF'
   <commit-message-here>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"`
2. Verify commit was successful

**Step 4: Push to Remote**
1. Get current branch name: `git branch --show-current`
2. Run `git push origin <current-branch>` or `git push -u origin <current-branch>` if branch doesn't exist on remote
3. Display: "Successfully pushed changes to remote branch: <branch-name>"
4. Exit

**Error Handling:**
- If no changes to commit: Display "No changes to commit" and exit
- If push fails (e.g., branch protection, conflicts): Display error and provide guidance
- If on main/master branch: Warn user that direct push to main is not recommended per CLAUDE.md

### Action: merge

**Step 1: Validate Branch Name**
1. Verify `branch-name` parameter is provided
2. If missing: Display error "Branch name is required for merge action. Usage: /git merge <branch-name>" and exit

**Step 2: Check Current Branch Status**
1. Run `git status` to get current branch name and check for uncommitted changes
2. Store current branch name for later use
3. Identify if there are:
   - Untracked files
   - Modified files (not staged)
   - Staged files (not committed)

**Step 3: Handle Uncommitted Changes**
If there are ANY uncommitted files (untracked, modified, or staged):
1. Display the current status output to user
2. Run `git add .` to stage all changes
3. Generate meaningful commit message based on changed files (analyze git status)
4. Run `git commit -m "$(cat <<'EOF'
   <commit-message-here>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"` following the commit message format
5. Run `git push origin <current-branch>` to push changes to remote
6. Display: "Committed and pushed changes before merge"

**Step 4: Fetch Latest from Remote**
1. Run `git fetch origin` to fetch all remote branches
2. Verify the source branch exists on remote: `git rev-parse --verify origin/<branch-name>`
3. If branch doesn't exist on remote: Display error "Branch '<branch-name>' does not exist on remote" and exit

**Step 5: Pull Latest Version of Source Branch**
1. Run `git pull origin <branch-name> --no-edit` to get latest version of source branch
2. Display: "Fetched latest changes from origin/<branch-name>"

**Step 6: Attempt Merge**
1. Run `git merge origin/<branch-name> --no-edit`
2. Check exit code to determine if merge was successful

**Step 7: Handle Merge Conflicts (if any)**
If merge has conflicts (exit code != 0 or `git status` shows conflicts):
1. Run `git status` to identify conflicted files
2. Parse output to get list of conflicted files
3. For EACH conflicted file:
   a. Display: "Conflict detected in: <file-path>"
   b. Ask user: "How would you like to resolve this conflict?
      1. Accept current branch version (--ours)
      2. Accept <branch-name> version (--theirs)
      3. Skip (manual resolution required)"
   c. Wait for user response (1, 2, or 3)
   d. **If user responds "1"**:
      - Run `git checkout --ours <file-path>`
      - Run `git add <file-path>`
      - Display: "Accepted current branch version for <file-path>"
   e. **If user responds "2"**:
      - Run `git checkout --theirs <file-path>`
      - Run `git add <file-path>`
      - Display: "Accepted <branch-name> version for <file-path>"
   f. **If user responds "3"**:
      - Display: "Skipped <file-path> - manual resolution required"
      - Continue to next file
4. After processing all files:
   - Check if any conflicts remain: `git status`
   - If conflicts remain: Display "Some conflicts require manual resolution. Please resolve and commit manually." and exit
   - If all resolved: Run `git commit --no-edit` to complete the merge
   - Display: "Merge completed with resolved conflicts"

**Step 8: Verify Merge Success**
1. If no conflicts OR all conflicts resolved:
   - Display: "Successfully merged origin/<branch-name> into <current-branch>"
   - Run `git log --oneline -3` to show recent commits
   - Display: "Merge complete. Latest commits shown above."
2. Exit

**Error Handling:**
- If source branch doesn't exist: Display error and exit
- If fetch/pull fails: Display error and provide guidance
- If merge is aborted: Run `git merge --abort` and inform user
- If conflicts cannot be auto-resolved: Provide clear instructions for manual resolution

## Implementation Notes

**Important Constraints from CLAUDE.md:**
- Main branch is protected - cannot push directly to main
- Always work on feature branches
- Follow commit message format with Claude Code attribution
- Merge latest changes from main when working on feature branch

**Validation Checks:**
- Validate `action` parameter is either "checkout", "push", or "merge"
- If `action=checkout`, validate `branch-name` parameter is provided
- If `action=merge`, validate `branch-name` parameter is provided
- Check if current branch is main/master when doing push action (warn but don't block)
- Verify git repository exists in current directory

**User Interaction:**
- Use clear, actionable prompts for Y/N questions
- Accept case-insensitive responses: Y, y, Yes, yes, YES / N, n, No, no, NO
- Display helpful error messages with next steps
- Show git command output for transparency

Execute the workflow based on the action parameter provided by the user.
