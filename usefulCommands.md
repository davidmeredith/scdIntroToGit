# Useful Commands

'''
git config --global user.name 'your name'
git config --global user.email 'email'


git status -s   # shows short status 
?|? - new / untracked files 
 |M - modified in working dir only
M|M - modified in staging area and work dir
M|  - modified in staging area only
A|  - new file added to staging area


git diff          # with no args shows diff between workDir/StageArea (ie changes not yet staged) 
git diff --staged # to compare stage area / last commit 
git diff --cached # alias for above
git log origin/master..HEAD  --oneline --stat  # view unpushed commit log (compare commits between origin/master to HEAD)
git diff origin/master..HEAD 


Unstage a staged file
git reset HEAD <file>...

Unmodify a modified file
git checkout --<file>


'''




