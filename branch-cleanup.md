---
argument-hint: "[--dry-run|--force]"
description: "Clean up stale and merged git branches"
model: claude-opus-4-5-20251101
allowed-tools: ["Bash", "Read", "AskUserQuestion"]
---

**If `$ARGUMENTS` is empty or not provided:**

Analyze branches and show what would be cleaned up (dry-run mode).

**Usage:** `/branch-cleanup [options]`

**Examples:**
- `/branch-cleanup` - Dry-run: show what would be deleted
- `/branch-cleanup --force` - Actually delete the branches
- `/branch-cleanup --dry-run` - Explicit dry-run mode

**Workflow:**
1. Fetch and prune remote-tracking references
2. Identify merged branches (safe to delete)
3. Find branches with deleted remotes
4. Detect stale branches (no commits in 30+ days)
5. Show summary and confirm before deletion

Proceed with dry-run analysis.

---

**If `$ARGUMENTS` is provided:**

Clean up stale and merged git branches based on the specified mode.

## Configuration

- **Mode**: `$ARGUMENTS`
  - `--dry-run` (default): Show what would be deleted without deleting
  - `--force`: Actually delete branches after confirmation

## Steps

1. **Sync Remote State**
   ```bash
   git fetch --prune origin
   ```
   This removes remote-tracking branches that no longer exist on the remote.

2. **Get Current Branch**
   ```bash
   git branch --show-current
   ```
   Never delete the current branch or protected branches (main, master, dev, develop).

3. **Identify Merged Branches**
   Find branches already merged into main/master/dev:
   ```bash
   git branch --merged main | grep -v "main\|master\|dev\|develop\|\*"
   ```
   These are safe to delete - their commits exist in the main branch.

4. **Find Branches with Deleted Remotes**
   Local branches whose remote tracking branch was deleted:
   ```bash
   git branch -vv | grep ': gone]' | awk '{print $1}'
   ```
   These typically indicate completed PRs that were merged and deleted on GitHub.

5. **Detect Stale Branches**
   Branches with no commits in 30+ days:
   ```bash
   git for-each-ref --sort=committerdate --format='%(refname:short) %(committerdate:relative)' refs/heads/
   ```
   Flag branches older than 30 days for review (but don't auto-delete).

6. **Categorize Results**

   Display summary:
   ```
   ## Branch Cleanup Analysis

   ### Safe to Delete (Merged)
   - feature/user-auth (merged into main)
   - fix/login-bug (merged into main)

   ### Remote Deleted (PR Completed)
   - feature/api-refactor (remote: gone)
   - fix/memory-leak (remote: gone)

   ### Stale (No Activity 30+ Days)
   - experiment/new-ui (45 days old) - REVIEW NEEDED
   - wip/cache-layer (60 days old) - REVIEW NEEDED

   ### Protected (Never Delete)
   - main
   - dev
   ```

7. **Execute Cleanup** (if `--force`)

   For safe deletions (merged + gone remotes):
   ```bash
   git branch -d <branch-name>  # Safe delete (refuses if not merged)
   ```

   For stale branches, ask user to confirm each one:
   - Show last commit message and date
   - Offer to use `-D` (force) if user confirms

8. **Post-Cleanup Summary**
   ```
   ## Cleanup Complete

   Deleted: 5 branches
   Skipped: 2 branches (user declined)
   Protected: 3 branches (main, dev, master)

   Run `git fetch --prune` periodically to keep clean.
   ```

## Safety Rules

- **Never delete**: main, master, dev, develop, release/*, production
- **Never delete**: Current branch (the one you're on)
- **Safe delete (-d)**: Merged branches, branches with deleted remotes
- **Confirm first (-D)**: Stale branches that aren't merged
- **Always dry-run first**: Show what would happen before doing it

## Notes

- This command is LOCAL-only by default - it doesn't delete remote branches
- To delete a remote branch: `git push origin --delete <branch>` (not done automatically)
- Configure auto-prune: `git config remote.origin.prune true`
- For team repos, coordinate cleanup to avoid deleting others' work
