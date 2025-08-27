# ST FlexiPatch Production Branch Management

## Updating from Upstream

### Prerequisites
Ensure the upstream remote is configured:
```bash
git remote add upstream https://github.com/bakkeby/st-flexipatch.git
```

### Step 1: Sync Master with Upstream
First, update the pristine `master` branch:
```bash
git checkout master
git fetch upstream master
git rebase --rebase-merges upstream/master
git push origin master
```

**Note**: There should be no conflicts here since `master` must remain pristine and unchanged.

### Step 2: Rebase Production Branch

**Warning**: This will likely produce conflicts that need to be resolved.

Rebase `prod` branch against the updated `master`:
```bash
git checkout prod
git rebase --rebase-merges master
```

**Note**: The `--rebase-merges` option preserves merge commits and ensures you don't have to resolve conflicts that were already resolved in previous merges.

#### Handling Conflicts
If merge conflicts occur:

1. **Resolve conflicts** in the affected files (use `git status` to see conflicted files)
   ```bash
   git status
   # Edit conflicted files to resolve conflicts
   # Optionally use mergetool:
   git mergetool
   ```

2. **Stage resolved files** and continue the rebase:
   ```bash
   git add resolved_file.c
   git rebase --continue
   ```

3. **If things go wrong**, you can always abort:
   ```bash
   git rebase --abort
   ```

### Step 3: Update Test Branch

After successfully updating `prod`, rebase the `test` branch:
```bash
git checkout test
git rebase --rebase-merges prod
```

Follow the same conflict resolution steps as above if needed.

### Step 4: Build and Test

After rebasing, rebuild and test the software:
```bash
./install.sh
```

This script will:
- Clean the build
- Remove existing `patches.h` and `config.h`
- Rebuild st
- Install it system-wide (requires sudo)
- Send USR1 signal to reload st (for Xresources)

### Step 5: Push Changes

Once everything is working:
```bash
git push --force-with-lease origin prod
git push --force-with-lease origin test
```

**Note**: Force push is required after rebasing. The `--force-with-lease` option is safer than `--force` as it ensures you don't overwrite any remote changes you haven't seen.

## Quick Reference

### Check Current Branch Status
```bash
git status
git log --oneline --graph --decorate -10
```

### View Differences
```bash
git diff master..prod  # See changes in prod vs master
git diff prod..test    # See changes in test vs prod
```

### Emergency Rollback
If an update breaks something critical:
```bash
git checkout prod
git reset --hard origin/prod  # Reset to last known good state
```

## Important Notes

1. **Never modify `master`** - It must remain identical to upstream/master
2. **Always test** after rebasing before pushing
3. **Keep backups** of your `config.h` and `patches.h` before updating
4. **Document conflicts** - Keep notes on recurring conflict resolutions for future updates
5. **Check patch compatibility** - Some patches may conflict; verify in `patches.def.h`
6. **Review config.mk** - Some patches require enabling specific libraries