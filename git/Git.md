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
