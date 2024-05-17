# env-config
## GIT
### GIT Config
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

### GIT Commands
1. Log
```
git log --oneline
```
2. [RefLog](https://www.atlassian.com/git/tutorials/rewriting-history/git-reflog)
> By default, reflogs keep track of each HEAD position throughout the last 90 days. Furthermore, the reflog history is exclusive to the repository and is not accessible remotely. Apart from branch tip reflogs, there is a separate reflog for the Git stash.
```
git reflog

```
* By default it's current branch, to specify a different branch or STASH
``` 
git reflog  otherbranch |  git reflog  stash

```
* use reference of these pointers in other git command
```
git diff stash@{0} otherbranch@{0}
```

* use timed reflog
```
# diff beftween main branch current head to 1 day ago
git diff main@{0} main@{1.day.ago} 
```


## HomeBrew 
brew install --cask dbeaver-community
brew install awscli
brew install kustomize
brew install node

### install psql command line utility
brew install libpq 
### install json processor
brew install jq


export JAVA_HOME=`/usr/libexec/java_home -v 11`

### VS Code config
https://github.com/speedingplanet/class-setup/blob/master/vs-code-extensions.md

