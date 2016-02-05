# Useful Commands

```
git config --global user.name 'your name'
git config --global user.email 'email'

# shows short status 
git status -s   
?|? - new / untracked files 
 |M - modified in working dir only
M|M - modified in staging area and work dir
M|  - modified in staging area only
A|  - new file added to staging area


git diff          # with no args shows diff between workDir/StageArea (ie changes not yet staged) 
git diff --staged # to compare stage area / last commit 
git diff --cached # alias for above

# view unpushed COMMITS (compare commits between origin/master to HEAD) 
git log origin/master..HEAD  --oneline --stat 
git diff origin/master..HEAD 


# Unstage a staged file
git reset HEAD <file>...

# Unmodify a modified file
git checkout --<file>


```




