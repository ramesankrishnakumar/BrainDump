# GIT
## GIT Config
1. Set username and email
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```
2. Execute git fetch and pull with `--prune` flag
```
$ git config --global fetch.prune true
```
3. Auto setup remote branch
```
git config --global --add --bool push.autoSetupRemote true
```

## Helpful Commands
### Log
```
git log --oneline
```
### [RefLog](https://www.atlassian.com/git/tutorials/rewriting-history/git-reflog)
> By default, reflogs keep track of each HEAD position throughout the last 90 days. Furthermore, the reflog history is exclusive to the repository and is not accessible remotely. Apart from branch tip reflogs, there is a separate reflog for the Git stash.
```
git reflog
```
1. By default it's current branch, to specify a different branch or STASH
``` 
git reflog  otherbranch |  git reflog  stash
```
2. Use reference of these pointers in other git command
```
git diff stash@{0} otherbranch@{0}
```

3. Use timed reflog
```
# diff beftween main branch current head to 1 day ago
git diff main@{0} main@{1.day.ago} 
```

### Diff
- find the diff between the stagged and the HEAD
```
git diff --stagged
```
- find the diff between commits
```
git diff commit-id-1 commit-id-2 | git diff commit-id-1..commit-id-2
```
- find the diff between branches
```
git diff branch1 branch2 | git diff branch1..branch2

```

### Stash
- set aside the current changes without commiting them, giving a message will help in identifying the stash
  - stash created in one branch and be applied to any branch
```
git stash -m "stashed work for XYZ project"
```
- make use of a stash
  - **pop** will remove the stash, **apply** will retain the stash   
```
git stash pop | git stash apply
```
- create a stash and apply the changes to a branch
  > cre­ates a new branch at the HEAD where the stash was cre­at­ed, checks that branch out, applies the stash to it, and then deletes the stash.
```
git stash branch branch-name
```
- stash untracked files
```
git stash -u | git stash --all
```

### Checkout
- Checkout a commit
```
git checkout <SHA> | git checkout HEAD~2
```
> If you checkout a commit sha directly, it puts you into a "detached head" state, which basically just means that the current sha that your working copy has checked out, doesn't have a branch pointing at it. If you haven't made any commits yet, you can leave detached head state by simply checking out whichever branch you were on before checking out the commit sha
```
git checkout <branch> | git switch - 
```
> If you did make commits while you were in the detached head state, you can save your work by simply attaching a branch before or while you leave detached head state:
```
  git checkout -b <new_branch> | git switch -c <new_branch>
```
- restore a file
```
git checkout -- <path/to/file> | git restore <path/to/file>
```

### Commit
- change a commit message of the latest commit 
```
git commit --amend -m "an updated commit message"
```
- add files to the newly added commit ** not pushed to remote **
```
# Edit hello.py and main.py
git add hello.py
git commit 
# Realize you forgot to add the changes from main.py 
git add main.py 
git commit --amend --no-edit
# --no-edit lets you keep the previous commit message
```

### Rebase



