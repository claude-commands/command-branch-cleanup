# command-branch-cleanup

A Claude Code slash command for cleaning up stale and merged git branches.

## Installation

```bash
# Clone to your preferred location
git clone git@github.com:claude-commands/command-branch-cleanup.git <clone-path>/command-branch-cleanup

# Symlink (use full path to cloned repo)
ln -s <clone-path>/command-branch-cleanup/branch-cleanup.md ~/.claude/commands/branch-cleanup.md
```

## Usage

```
/branch-cleanup              # Dry-run: show what would be deleted
/branch-cleanup --force      # Actually delete branches
/branch-cleanup --dry-run    # Explicit dry-run mode
```

## What it does

1. Fetches and prunes remote-tracking references
2. Identifies branches merged into main/dev
3. Finds branches with deleted remote tracking
4. Detects stale branches (30+ days inactive)
5. Categorizes and confirms before deletion

## Output Format

```
## Branch Cleanup Analysis

### Safe to Delete (Merged)
- feature/user-auth (merged into main)
- fix/login-bug (merged into main)

### Remote Deleted (PR Completed)
- feature/api-refactor (remote: gone)

### Stale (No Activity 30+ Days)
- experiment/new-ui (45 days old) - REVIEW NEEDED

### Protected (Never Delete)
- main, dev
```

## Safety Features

| Category | Action |
|----------|--------|
| Merged branches | Safe delete (`-d`) |
| Remote deleted | Safe delete (`-d`) |
| Stale branches | Confirm each (`-D`) |
| Protected branches | Never delete |

## Protected Branches

These are never deleted:
- `main`, `master`
- `dev`, `develop`
- `release/*`, `production`
- Current branch

## Requirements

- Git repository
- Claude Code with Sonnet model access

## Updates

```bash
cd <clone-path>/command-branch-cleanup && git pull
```
