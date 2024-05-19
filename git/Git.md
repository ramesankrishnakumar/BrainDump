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
- pull files from a different branch
fetch the file **server.json** from the branch **john** to the current branch
```
git checkout john -- data/server.json
```
- go back to the previous branch
```
git checkout -
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
![rebase](git_rebase.svg)
> The primary reason for rebasing is to maintain a linear project history. For example, consider a situation where the main branch has progressed since you started working on a feature branch. You want to get the latest updates to the main branch in your feature branch, but you want to keep your branch's history clean so it appears as if you've been working off the latest main branch. This gives the later benefit of a clean merge of your feature branch back into the main branch.

- rebase from main to the working branch
```
git rebase main
```
<p>This automatically rebases the current branch onto main, **refer the image**</p>

- edit / reword / squash / fixup previous commits
**Go to a commit that is parent of the commit to which you want to modify**
> Note that the commits are listed from earliest to latest, so the most recent commits are shown at the bottom of the list. This is the opposite direction to the way most git graph visualizations show the most recent commits at the top of the graph.
> The difference between squash and fixup, is that squash let's you edit the resulting commit log message; fixup, on the other hand, defaults to using the previous commit's log message.
```
git rebase -i HEAD~5
```
```
pick 43f8707f9 fix: update dependency json5 to ^2.1.1
pick cea1fb88a fix: update dependency verdaccio to ^4.3.3
pick aa540c364 fix: update dependency webpack-dev-server to ^3.8.2
pick c5e078656 chore: update dependency flow-bin to ^0.109.0
pick 11ce0ab34 fix: Fix spelling.

# Rebase 7e59e8ead..11ce0ab34 onto 7e59e8ead (5 commands)
```

### Reset
![reset](git_reset.svg)
> You should never use git reset  when any snapshots after have been pushed to a public repository. After publishing a commit, you have to assume that other developers are reliant upon it.

> If you need to fix a public commit, the git revert command was designed specifically for this purpose.

> The default invocation of git reset has implicit arguments of --mixed and HEAD. This means executing git reset is equivalent to executing git reset --mixed HEAD. In this form HEAD is the specified commit. Instead of HEAD any Git SHA-1 commit hash can be used.

- hard
Changes are made to the commit ref, staging area and working directory. The branch is reset to the state of the commit (HEAD in this case), changes in staging are and working directory are lost
```
git reset --hard
```
- mixed which is the default
Changes are made to the commit ref and the staging area. The commit ref is modifed to the specified commit (HEAD in this case) and the changes in the staging area are moved to the working directory ref
```
git reset | git reset --mixed
```
- soft
Changes are made only to commit ref, staging area and working directory are left untouched
```
git reset --soft
```

### Cherry Pick

### Bisect

### Aliases






