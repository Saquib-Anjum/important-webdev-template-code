# 🚨 Essential Git Commands for Risky Situations

This guide contains important Git commands to fix mistakes, undo changes, and recover safely. Keep it handy in your repo.

---

## 🔄 Undoing Commits

### Delete the Last Commit (Keep Changes in Working Directory)
```bash
git reset --soft HEAD~1
```
- Removes last commit but keeps your changes staged.

### Delete the Last Commit (Discard Changes)
```bash
git reset --hard HEAD~1
```
- Removes last commit **and** discards changes.

### Delete Multiple Commits (e.g., last 3 commits)
```bash
git reset --hard HEAD~3
```
- Deletes last 3 commits permanently.

---

## 🔒 Removing Sensitive Data (Passwords, Keys, etc.)

### Rewrite Git History to Remove a File from All Commits
```bash
git filter-branch --force --index-filter "git rm --cached --ignore-unmatch <file>" --prune-empty --tag-name-filter cat -- --all
```
- Removes a file (like `.env` or password file) from entire history.

### After History Rewrite, Force Push
```bash
git push origin --force --all
git push origin --force --tags
```
⚠️ Use with caution. This changes history for everyone.

---

## 🕑 Reverting Changes

### Revert a Specific Commit (without deleting history)
```bash
git revert <commit-hash>
```
- Safely undoes a commit by creating a new commit.

### Restore a Deleted File
```bash
git checkout HEAD <file>
```
- Brings back the last committed version of the file.

---

## 🧹 Cleaning Working Directory

### Discard All Local Changes (Unstaged)
```bash
git checkout -- .
```
- Restores all files to last commit state.

### Remove Untracked Files/Folders
```bash
git clean -fd
```
- Deletes files not tracked by Git.

---

## 🔀 Branch Recovery & Reset

### Reset a Branch to Remote (Discard Local Commits)
```bash
git fetch origin
git reset --hard origin/<branch>
```
- Makes branch identical to remote.

### Recover Lost Commits (Using Reflog)
```bash
git reflog
git checkout <commit-hash>
```
- Shows all actions and lets you recover lost commits.

---

## 🛑 Stash Changes (Before Risky Actions)

### Save Work in Progress
```bash
git stash
```

### Apply Stashed Changes Back
```bash
git stash pop
```

---

## ⚡ Force Push (Careful)

### Overwrite Remote with Local
```bash
git push origin <branch> --force
```
- Use only when you’re sure remote history should be replaced.

---

## 📌 Extra Tips
- Always double-check before using `--hard` or `--force`.
- Use `git log --oneline` to see commits quickly.
- Backup your branch before risky actions:  
  ```bash
  git branch backup-branch
  ```

---

✅ Keep this file in your repo as a quick rescue guide.
