An introduction to Git via demos 
=================================
Tip: To view this markdown file (.md) as a webpage, install a markdown viewer/plugin into your browser.


demo1 - Init repo from existing files, add .gitignore, commit
-------------------------------------------------------------
* Create a Git repo from an existing project.
* Create some files that will be added to .gitignore. 
* Add all files to staging area (index), commit and repeat. 
* Note ignored files are not included, and the linear history and the location of HEAD pointer.  

```bash
mkdir demo1; cd demo1      # create demo subdir
vim index.html             # copy content from above
vim page1.html             # copy content from above 
git status                 # note not a git repo yet 
git init                   # create repo from existing files 
git status                 # note Untracked files not yet staged
mkdir private              # create  private dir and password before we add to repo
touch private/password.txt
vim .gitignore             # create .gitignore add private dir, *.tmp, *~, *.swp
git status --ignored       # note .gitignore added and private dir are ignored (--ignored to list ignored files too)
git add .                  # add all 
git status                 # note files to be committed
git commit                 # initial commit             
git status                 # note working directory clean
vim index.html             # make some small modifications 
vim page1.html             # make some small modifications 
git diff                   # show modified lines between workDir/Index
git add .
git diff --staged          # show modified lines between index/lastCommit
git commit -m "first modifications" 
git status                 # note clean working dir
git log --oneline --graph --decorate --all # note HEAD-pointsTo->master-pointsTo->hash 
git show <commitID>        # to show what was changed in commit
git help commit            # fire up help in browser
git 
```



demo4 - Three Way Merge then FastForward 
-----------------------------------------
* Create a forked history
* Merge footer into master using 3way merge
* Bring footer up to date with master
* Delete footer 'topic branch' 

```bash
git checkout -b footer            # create and checkout footer branch
vim index.html                    # un-comment footer
vim page1.html                    # un-comment footer 
git commit -a -m "adding footer"  # stage all modified files and commit 
git log --oneline --graph --decorate --all   # at this point, we have not forked, but note master is behind footer 
	* cf8dcae (HEAD, footer) adding footer
	* 3ffa711 (master) fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout master
vim index.html       # note footer still commented out, add a comment 
git commit -a -m "add comment"   # We fork here - master moves on independently of footer as below

git log --oneline --graph --all --decorate
	* 7a851ae (HEAD, master) adding comment
	| * cf8dcae (footer) adding footer
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git merge footer
	Auto-merging index.html
	Merge made by the 'recursive' strategy.
	 index.html | 4 ++--
	 page1.html | 4 ++--
	 2 files changed, 4 insertions(+), 4 deletions(-)
 
more index.html                  # note footer and comment integrated 
git checkout footer
git merge master                 # note fast forward merge 
	Updating cf8dcae..a5d3680
	Fast-forward
	 index.html | 2 +-
	 1 file changed, 1 insertion(+), 1 deletion(-)

git log --oneline --graph --all --decorate  # note footer, master up to date
git checkout master
git branch -d footer    # delete local footer branch 
```

demo6 - Merge Conflict
-----------------------
* Create a forked history with changes to same lines in two different versions of same file. 
* 3 way merge produces conflict
* Fix merge conflicts 

```bash
git log --oneline --graph --decorate --all  # note fork start position (footer in footer and text in master)
	* 568438c (HEAD, master) adding paragraph
	| * 33991ba (footer) adding footer
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

vim index.html          # add 'Version 1 body text' below <div>index body</div>
git commit -a -m "adding v1 body text"
git checkout footer
vim index.html          # add 'Version 2 body text' below <div>index body</div>
git commit -a -m "adding v2 body text"
git log --oneline --graph --decorate --all   # note extended forks 
	* 6635ab1 (HEAD, footer) adding v2 body text
	* 33991ba adding footer
	| * 45c759f (master) adding body text
	| * 568438c adding paragraph
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout master
git merge footer      # note conflict
	Auto-merging index.html
	CONFLICT (content): Merge conflict in index.html
	Automatic merge failed; fix conflicts and then commit the result.
git status
	On branch master
	You have unmerged paths.
	  (fix conflicts and run "git commit")

	Changes to be committed:

		modified:   page1.html

	Unmerged paths:
	  (use "git add <file>..." to mark resolution)

		both modified:   index.html

vim index.html       # fix merge conficts
git add index.html   # re-stage
git commit -m "merge conflict fixed"  
less index.html      # note conflict resolved and footer is merged 
```

demo7 stashing
--------------
* Try to checkout master - prevented
* Note local changes 
* Stash local changes 
* checkout master
* checkout dev
* pop local changes back into dev

```bash
git branch
	* dev
	  master
git checkout master
	error: Your local changes to the following files would be overwritten by checkout:
		index.html
	Please, commit your changes or stash them before you can switch branches.
	Aborting

git status             # notice locally modified index and untracked page2 file
git diff index.html     # note local change <!-- a change to stash-->
	diff --git a/index.html b/index.html
	index c4b60ba..ded398c 100644
	--- a/index.html
	+++ b/index.html
	@@ -14,6 +14,7 @@
		     <a href="page1.html">page1</a>
		 </div>
		 Added paragraph in master
	+        <!-- a change to stash-->

git stash -u         # stash the local changes ('-u' for to stash untracked file page2)
less index.html      # note stashing has reverted local changes (index file has rolled-back to last commit and page2 is removed) 
ls
index.html  page1.html  private

git stash list       # note the stashed changes in stash@{0} (WIP = work in progress) 
git checkout master  # since local changes are stashed, we can now move between branches once again
git checkout dev     # 
git stash pop        # pop stashed changes off stash back into dev branch  
git stash list       # note stash is now empty 
git status           # note stashed changes have been restored into dev
git checkout master  # as before we CANT now switch to master again  
      error: Your local changes to the following files would be overwritten by chec

```

* Error occurs when you have a modified local file and the branch that you want to switch to has a diverged version of the same file.
* Q. What if an urgent bug fix comes in and we need to work on the master branch?
* A. We can either commit these local changes in dev (but we are not ready to do this yet) or we can stash our local changes:




```bash
git config --global http.proxy wwwcache.dl.ac.uk:8080
git config --global http.proxy wwwcache.dl.ac.uk:8080

git config --global --unset http.proxy
git config --global --unset https.proxy
```

