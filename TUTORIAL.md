# DataSpecDashBoard — Collaboration Tutorial

A step-by-step guide for working with collaborators on this repository.

---

## 1. Invite a Collaborator

1. Go to your repo on GitHub: `https://github.com/YimengShang/DataSpecDashBoard`
2. Click **Settings** → **Collaborators** → **Add people**
3. Search by their GitHub username or email
4. They will receive an email invitation — they must accept it before they can push

---

## 2. Collaborator Setup (First Time)

The collaborator clones the repo to their machine:

```bash
git clone https://github.com/YimengShang/DataSpecDashBoard.git
cd DataSpecDashBoard
```

---

## 3. Recommended Workflow — Feature Branches + Pull Requests

Never work directly on `main`. Each change lives on its own branch.

### Collaborator: make a change

```bash
git checkout -b my-feature-branch        # create and switch to new branch
# ... make edits to data-spec-builder.html ...
git add data-spec-builder.html
git commit -m "Add XYZ section to variable definitions"
git push origin my-feature-branch        # push branch to GitHub
```

### Collaborator: open a Pull Request

1. Go to the repo on GitHub
2. Click **Compare & pull request** (banner appears after push)
3. Add a title and description of what changed and why
4. Click **Create pull request**

### You (owner): review and merge

1. Go to the repo → **Pull requests** tab
2. Click the PR to review the diff (green = added, red = removed)
3. Leave inline comments on specific lines if needed
4. Options:
   - **Merge pull request** — accept all changes
   - **Request changes** — send back for revision
   - **Close pull request** — reject without merging

---

## 4. Cherry-Pick — Accept Only Specific Commits

If a collaborator pushed multiple commits but you only want some:

```bash
git fetch origin                                   # download their changes
git log origin/their-branch --oneline              # see their commits

# example output:
# a1b2c3d Fix typo in exclusion criteria section
# e4f5g6h Add new feasibility check options       <-- you want this one
# i7j8k9l Experimental UI change                  <-- you don't want this

git cherry-pick e4f5g6h                            # apply only that commit
```

---

## 5. Stay in Sync — Pull Before You Work

Always pull the latest changes before starting work to avoid conflicts:

```bash
git pull origin main
```

If there are conflicts, Git will mark them in the file:

```
<<<<<<< HEAD
your version of the line
=======
their version of the line
>>>>>>> their-branch
```

Edit the file to keep the correct version, remove the markers, then:

```bash
git add data-spec-builder.html
git commit -m "Resolve merge conflict in spec builder"
```

---

## 6. Common Commands Reference

| Task | Command |
|------|---------|
| Get latest changes | `git pull origin main` |
| Create a new branch | `git checkout -b branch-name` |
| See what changed | `git diff` |
| Stage a file | `git add filename` |
| Commit | `git commit -m "message"` |
| Push a branch | `git push origin branch-name` |
| List branches | `git branch -a` |
| Switch branch | `git checkout branch-name` |
| Pick one commit | `git cherry-pick <hash>` |

---

## 7. Protect the Main Branch (Optional but Recommended)

Prevent anyone (including yourself) from pushing directly to `main` without a PR review:

1. GitHub repo → **Settings** → **Branches**
2. Click **Add branch protection rule**
3. Branch name pattern: `main`
4. Check **Require a pull request before merging**
5. Check **Require approvals** (set to 1)
6. Click **Save changes**

---

## Tips

- Write clear commit messages — explain *why*, not just *what*
- Keep PRs small and focused on one change at a time
- Use the PR description to explain what was changed and how to test it
- Review the diff carefully before merging — once merged it's in `main`
