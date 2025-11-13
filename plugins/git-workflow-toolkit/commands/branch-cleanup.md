---
description: Clean up merged and stale branches locally and remotely
allowed-tools: Bash(git:*), Read
argument-hint: [--dry-run] [--remote] [--force]
---

# Branch Cleanup

## Context

- Current branch: !`git branch --show-current`
- Local branches: !`git branch --list | wc -l`
- Remote branches: !`git branch -r | wc -l`
- Merged branches: !`git branch --merged | grep -v "^\*" | grep -v "main" | grep -v "master" | grep -v "develop" | wc -l`

## Your Task

Identify and clean up merged and stale branches to maintain a clean repository.

### Arguments

- `--dry-run` (optional): Show what would be deleted without actually deleting
- `--remote` (optional): Also clean up remote branches
- `--force` (optional): Delete unmerged branches (use with caution)

### Steps

1. **Fetch latest changes**:
   ```bash
   git fetch --prune
   ```

2. **Identify merged branches**:
   ```bash
   git branch --merged | grep -v "^\*" | grep -v "main" | grep -v "master" | grep -v "develop"
   ```

3. **Identify stale branches** (no commits in 30+ days):
   ```bash
   git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'
   ```

4. **Review branches to delete**:
   - Merged feature branches
   - Merged bugfix branches
   - Stale branches (no activity in 30+ days)
   - Branches with naming like `test-*`, `tmp-*`, `wip-*`

5. **Delete local merged branches**:
   ```bash
   git branch --merged | grep -v "^\*" | grep -v "main" | grep -v "master" | grep -v "develop" | xargs git branch -d
   ```

6. **Delete remote merged branches** (if --remote flag):
   ```bash
   git branch -r --merged | grep -v "main" | grep -v "master" | grep -v "develop" | sed 's/origin\///' | xargs -I {} git push origin --delete {}
   ```

7. **Report cleanup results**:
   - Number of branches deleted
   - List of deleted branches
   - Remaining active branches
   - Disk space saved (if significant)

### Examples

**Dry run to preview**:
```
/branch-cleanup --dry-run
# Shows what would be deleted without actually deleting
```

**Clean up local branches**:
```
/branch-cleanup
# Deletes merged local branches
```

**Clean up local and remote**:
```
/branch-cleanup --remote
# Deletes merged branches locally and on remote
```

**Force delete unmerged branches**:
```
/branch-cleanup --force
# Deletes even unmerged branches (dangerous!)
```

### Safety Checks

**Protected branches** (never delete):
- `main` / `master`
- `develop` / `development`
- `staging` / `production`
- Current branch (checked out)

**Confirmation prompts**:
- Show list of branches to delete
- Ask for confirmation before deleting
- Warn about unmerged branches
- Confirm before remote deletion

### Output Format

```
🧹 Branch Cleanup Report
========================

Fetching latest changes...
✓ Fetched from origin

Analyzing branches...
Found 15 local branches
Found 23 remote branches

Merged branches (safe to delete):
  ✓ feature/user-auth (merged 5 days ago)
  ✓ bugfix/login-error (merged 2 weeks ago)
  ✓ feature/dashboard (merged 1 month ago)

Stale branches (no activity in 30+ days):
  ⚠ feature/old-experiment (last commit 45 days ago)
  ⚠ test-branch (last commit 60 days ago)

Unmerged branches (keep):
  → feature/new-feature (active, 2 days old)
  → bugfix/critical-fix (active, 1 day old)

Deleting merged branches...
  ✓ Deleted feature/user-auth
  ✓ Deleted bugfix/login-error
  ✓ Deleted feature/dashboard

Summary:
  • Deleted: 3 branches
  • Kept: 2 active branches
  • Protected: 2 branches (main, develop)
  • Remaining: 4 local branches

💡 Tip: Run with --remote to also clean up remote branches
```

### Best Practices

**Regular cleanup**:
- Run weekly or after major releases
- Clean up after merging PRs
- Delete feature branches immediately after merge
- Keep repository lean

**Before cleanup**:
- Ensure all important work is merged
- Check with team about shared branches
- Backup if unsure
- Use --dry-run first

**After cleanup**:
- Verify important branches remain
- Check CI/CD still works
- Update documentation if needed
- Notify team of cleanup

### Recovery

If branch deleted by mistake:
```bash
# Find deleted branch commit
git reflog

# Recreate branch
git branch branch-name <commit-hash>

# Or restore from remote
git checkout -b branch-name origin/branch-name
```
