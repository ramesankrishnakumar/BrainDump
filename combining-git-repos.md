# Combining Multiple Git Repos in One Folder

Goal: organize several separate repos (all owned by me, some private to others)
into one folder for context, while keeping them independent. Always track latest
`master`, and be able to contribute from either the workspace or the individual repos.

## Recommended approach: Git submodules tracking a branch

Each submodule keeps its own GitHub remote and access control. The workspace is
just a wrapper that references them. **The workspace does NOT need its own GitHub
remote** — keep it purely local unless you want one-command reproducible clones.

### One-time setup

```bash
cd ~/Projects/workspace
git init

# --branch makes the submodule follow a branch instead of pinning a commit
git submodule add --branch master git@github.com:you/repo-a.git
git submodule add --branch master git@github.com:you/repo-b.git

# Make `git pull` also advance submodules, and default updates to merge
git config -f .gitmodules submodule.repo-a.update merge
git config submodule.recurse true

# Optional: only if you want to snapshot versions
git commit -m "Add workspace submodules tracking master"
```

No remote required. Each repo still pushes/pulls to its own GitHub remote from
inside its subfolder.

### Day-to-day workflow

**Sync latest master everywhere:**
```bash
git submodule update --remote --merge
```

**Contribute (from inside the workspace):**
```bash
cd repo-a
git checkout master            # IMPORTANT: avoid detached HEAD before committing
# ...edit...
git add . && git commit -m "fix"
git push                       # pushes to repo-a's own remote

# Optional: record the new commit pointer in the workspace
cd ..
git add repo-a && git commit -m "Bump repo-a"
```

**Contribute (from a standalone clone elsewhere):** work and push as normal —
same repo. Then back in the workspace run `git submodule update --remote` to pull
those changes in.

### The one gotcha: detached HEAD

A fresh submodule checkout lands on a *commit*, not the branch. Always
`git checkout master` inside a submodule before committing, or `git push` has
nothing to push.

## When to give the workspace a remote (optional)

Only worth it if you want to reproduce the exact set + versions on another machine:

```bash
git clone --recurse-submodules <workspace-url>
```

Otherwise skip it.

## Even simpler alternative

If you never need the one-command bundle clone, skip submodules entirely: just a
plain parent folder with each repo cloned inside it. Submodules only earn their
keep once you want reproducible clones or version snapshots.

```
~/Projects/workspace/
├── repo-a/   (own .git, own remote)
├── repo-b/
└── repo-c/
```

## Quick reference

| Want | Do |
| --- | --- |
| Latest master everywhere | `git submodule update --remote --merge` |
| Contribute | edit → `git checkout master` → commit → push *inside the submodule* |
| Snapshot versions | commit the pointer bump in the workspace |
| Reproducible clone elsewhere | give workspace a remote, `git clone --recurse-submodules` |
