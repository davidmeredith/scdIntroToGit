An introduction to Git via demos 
=================================
Tip: To view this markdown file (.md) as a webpage, install a markdown viewer/plugin into your browser. 
* Start each demo in a new subdir. 
* Demos are consecutive, so after each demo, it is recommended to copy and rename the current demo's subdir to start the next lesson. In doing this, you will have a working directory with the correct file/repository state to (re)start the lesson if you get into a pickle. 
* Two seed files to start the first demo below. 

### index.html

``` html
<!DOCTYPE html>

<html>
    <head>
        <title>index page</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <div>index body</div>

        <div id="mymenu">
            <a href="page1.html">page1</a> 
        </div> 


        <!--
        <div id="footer">
            Courtesy STFC
        </div>
        -->
        
    </body>
</html>
```

### page1.html

``` html
<!DOCTYPE html>

<html>
    <head>
        <title>index page</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <div>page 1 body</div>

        <div id="mymenu">
            <a href="page1.html">page1</a> 
        </div> 


        <!--
        <div id="footer">
            Courtesy STFC
        </div>
        -->
        
    </body>
</html>
```

demo1 - Create repo from existing files, add .gitignore
--------------------------------------------------------
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
git add . 
git commit -m "first modifications" 
git status                 # note clean working dir
git log --oneline --graph --decorate --all # note HEAD-pointsTo->master-pointsTo->hash 
```

demo2 - Delete/Rename files, Checkout snapshot 
----------------------------------------------
* Delete a versioned file and commit (but leave in working dir), note outupt of `git status`.  
* Checkout a historical commit (detached-HEAD-state), note working dir files are updated to match the commit snapshot and the deleted page is restored (note we are not checking out another branch, this comes later). 
  * `git checkout <commitID>`  (where `<commitID>` is a hash or reference to a hash, e.g. `HEAD~2`)
  * Git changes HEAD to point to the commitID putting git in 'detached-HEAD-state' 
  * Populates Index with the snapshot to reflect the commitID snapshot  
  * Copies the contents of the Index into your Working Directory (is safe - any local modifications to the files in the working tree are kept, so that the resulting working tree will be the state recorded in the commit plus the local modifications). 
* Checkout master branch, note HEAD pointer is moved and local files are updated (page2 removed). 

```bash
cp -r demo1 demo2
cd demo2
cp page1.html page2.html   
git status                   # note page2 not staged 
git add page2.html           # add/stage
git status                   # page2 now staged 
git commit                   # commit (we will delete this file later on) 
git status 
git rm --cached page2.html   # schedule page2 for deletion, --cached leaves the file as untracked in workingdir
git status                   # page2 staged for remove but left in working dir
git commit
git status                   # page2 untracked 
rm page2.html                # can safely delete when ready 
git status
git log --oneline --graph --decorate  # note linear history all on master
git checkout HEAD~1          # detach head to checkout historical snapshot - note page2.html is restored
git checkout master          # switch back to master and re-attach head - note page2.html removed 
```

demo3 - Branching1: Linear history and 'FastForward' merge
------------------------------------------------------------
* Create a topic branch 'bugFix' and check it out, note we are on new branch. 
* Fix a bug in file and stage/commit, note bugFix branch has moved ahead of master. 
* Checkout master and note the bug fix is not yet merged. 
* Merge bugFix into master for Fast-forward merge. 
* Show the patch introduced by bugFix merge. 
* Delete the bugFix topic branch. 

```bash
git branch             # note we are on 'master' branch
git checkout -b bugFix # create bugFix branch and check it out
git branch             # note we are now on bugFix indicated by * 
vim page1.html         # fix bug in menu link (<a href=> should point to index.html)  
git status             # note page1 is tracked and modified but not yet staged for commit
git add page1.html     
git commit 
git log --oneline --graph --decorate
	* 3ffa711 (HEAD, bugFix) fixed bug in page1      # note bugFix branch has now moved ahead of master
	* 7af7c7a (master) removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout master             # updates (unmodified) wrkDir files (safe - will not overwrite local changes)
less page1.html                 # note that the bug fix is missing (link is to page1.html not index)
git merge bugFix                # merge bugFix changes into master, note fast forward 
Updating 7af7c7a..3ffa711
Fast-forward
 page1.html | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

git log --oneline --graph --decorate -all   # note HEAD, master, bugFix all on latest commit 
	* 3ffa711 (HEAD, master, bugFix) fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git show 3ffa                         # show patch of commit (menu link modified)
git branch -d bugFix                  # delete bugFix, no longer needed
git log --oneline --graph --decorate -all   # note deleted bugFix branch
```
(lesson2 - Branching in Git - Visually summarises above) 


demo4 - Branching2: Forked history and 3 way 'recursive' merge 
--------------------------------------------------------------
* Create/checkout footer branch, add footer code and commit.
* Checkout master and modify/commit index.html file to produce forked history. 
* Note changes in footer branch not yet merged. 
* Merge footer into master and note recursive merge and master moves ahead of footer branch. 
* Delete footer topic branch. 

```bash
git checkout -b footer        # create and checkout footer branch
vim index.html                # un-comment footer
vim page1.html                # un-comment footer 
git commit -a                 # stage all modified files and commit 
git log --oneline --graph --decorate --all   # at this point, we have not forked, but note master is behind footer 
git log --oneline master..footer             # show commits just in footer branch (since master..until footer) 
git checkout master
vim index.html       # note footer still commented out, add a new paragraph 
git commit -a -m "add paragraph"   # We fork here - master moves on independently of footer as below
git log --oneline --graph --decorate --all
	* 0143152 (HEAD, master) adding new paragraph
	| * 79915fd (footer) adding footer
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

less index.html       # note that footer is not yet integrated
less page1.htm        # note that footer is not yet integrated
git merge footer      # merge footer into master - note 'recursive' strategy (3 way branch tip merge) 
Auto-merging index.html       
	Merge made by the 'recursive' strategy.
	 index.html | 2 --
	 page1.html | 2 --
	 2 files changed, 4 deletions(-)

git log --oneline --graph --decorate --all     # master is now ahead of footer 
	*   4bc768b (HEAD, master) Merge branch 'footer'
	|\
	| * 79915fd (footer) adding footer
	* | 0143152 adding new paragraph
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git branch -d footer    # delete branch, note the branch (pointer) is removed from graph (history not affected)
git log --oneline --graph --decorate --all
	*   4bc768b (HEAD, master) Merge branch 'footer'
	|\
	| * 79915fd adding footer
	* | 0143152 adding new paragraph
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit
```	
(lesson3 - Merging in Git - Visually summarises above)


demo5 - Rebasing: Flatten a fork to create a flat history   
---------------------------------------------------------
* Create/checkout updateFooterFix branch, update footer code in both html files and commit. 
* Checkout master and change/commit index.html file (add 2nd paragraph) to create forked history. 
* Rebase 'updateFooterFix' onto the end of 'master' - note fork is flattened. 
* Bring master pointer up to date - note fastForward merge. 
* Delete updateFooterFix branch 

```bash
git checkout -b updateFooterFix
vim index.html      # update footer (add text 'and SCD')
vim page1.html      # update footer (add text 'and SCD')
git add .
git commit -m "updating footer"   
git checkout master # head back to master for some other work 
vim index.html      # add a 2nd paragraph 
git commit -a -m "adding 2nd paragraph in master"    # created forked history 
git log --graph --oneline --decorate --all
	* 5fa6720 (HEAD, master) adding 2nd paragraph in master
	| * 8097e45 (updateFooterFix) updating footer
	|/
	*   4bc768b Merge branch 'footer'
	|\
	| * 79915fd adding footer
	* | 0143152 adding new paragraph
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout updateFooterFix    
git rebase master                  # rebase current branch onto master - note updateFooterFix has been rebased flattend 
git log --oneline --graph --decorate --all    # note 'updateFooterFix' fork has been flattenend and is ahead of master 
	* 142f850 (HEAD, updateFooterFix) updating footer
	* 5fa6720 (master) adding 2nd paragraph in master
	*   4bc768b Merge branch 'footer'
	|\
	| * 79915fd adding footer
	* | 0143152 adding new paragraph
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout master         # need to bring master up to date 
git merge updateFooterFix   # as history is linear, merging updateFooterFix into master simply fast-forwards master
	Updating 5fa6720..142f850
	Fast-forward
	 index.html | 2 +-
	 page1.html | 2 +-
	 2 files changed, 2 insertions(+), 2 deletions(-)

git log --oneline --graph --decorate --all
	* 142f850 (HEAD, updateFooterFix, master) updating footer
	* 5fa6720 adding 2nd paragraph in master
	*   4bc768b Merge branch 'footer'
	|\
	| * 79915fd adding footer
	* | 0143152 adding new paragraph
	|/
	* 3ffa711 fixed bug in page1
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit
git branch -d updateFooterFix            # delete topic branch 
```
(lesson 4 objective - Rebasing) 


demo6 - Dealing with Merge Conflicts
-------------------------------------
* Create/checkout a dev branch, modify index.html by updating the text/paragraphs, commit.    
* Checkout master and modify same index.html file by updating same text, commit (creating new fork in history). 
* Merge dev into master, creates a CONFLICT. 
* Fix conflict markers in file, add/stage and commit to fix conflict. 

```bash
git checkout -b dev
vim index.html            # modify both paragraphs (add text 'modified in DEV') 
git commit -a -m "made some changes in DEV"
git checkout master
vim index.html            # modify same/both paragraphs (add text 'modified in MASTER') 
git commit -a -m "made some changes in MASTER"
git log --oneline --graph --decorate --all         # note new fork created 
	* 31b6416 (HEAD, master) modified in sentence and paragraph in MASTER
	| * dc0cb25 (dev) modified body and paragraph in DEV
	|/
	* 2156bf0 updating footer to credit SCD
	* 7beb526 adding second paragraph in master
	*   dcb762b Merge branch 'footer'
	|\
	| * 3b1a4d9 adding footer
	* | b172f8d adding body content
	|/
	* d8b5ce6 fixed bug menu link
	* 0611814 removing page2, too early
	* 09cbe2f adding page2
	* 2ffc846 initial import

git checkout master    # make sure we are on master
git merge dev          # attempt to merge dev into master - conflict
	Auto-merging index.html
	CONFLICT (content): Merge conflict in index.html
	Automatic merge failed; fix conflicts and then commit the result.
vim index.html         # fix confict markers
git add .              # indicates merge is fixed
git commit             # note default message indicates fix of merge conflict 
git log --oneline --graph --decorate --all              # note we are up to date again 
	*   6e5753c (HEAD, master) Merge branch 'dev'
	|\
	| * dc0cb25 (dev) modified body and paragraph in DEV
	* | 31b6416 modified in sentence and paragraph in MASTER
	|/
	* 2156bf0 updating footer to credit SCD
	* 7beb526 adding second paragraph in master
	*   dcb762b Merge branch 'footer'
	|\
	| * 3b1a4d9 adding footer
	* | b172f8d adding body content
	|/
	* d8b5ce6 fixed bug menu link
	* 0611814 removing page2, too early
	* 09cbe2f adding page2
	* 2ffc846 initial import
git merge master dev   # merge master into dev (fast forwards dev) 
```

demo7 - Stashing  
-----------------------------------------------------------------
* Why stashing? - Stashing of changes in local working dir allows you to checkout a diverged branch 
1. Ensure HEAD, master and dev are all on latest commit 
2. Create some local changes and an untracked file
3. Note you can still switch between branches because branches have not yet diverged 
4. Commit a modified file to move dev branch ahead of master 
5. Locally modify same file (file is now 'ahead' of master version + locally modified) 
6. Prevented checkout of different branch 

```bash
git checkout dev
vim index.html             # add some new text e.g. 'move dev ahead of master'
cp page1.html page2.html   # create new local file
git status                 # note local change and untracked file 
git add index.html         # now stage just the modified index.html 
git commit -m "moving dev ahead of master"   # commit to move dev ahead of master 
vim index.html             # modify same file (add new text 'change to stash'), file is locally modified + 'ahead' of master  
git checkout master        # now try to checkout master branch - we get error 
	error: Your local changes to the following files would be overwritten by checkout
```

* Error occurs when you have a **modified local file** and the branch that you want to switch to has a **diverged version of the same file**.  
* Q. What if an urgent bug fix comes in and we need to work on the master branch? 
* A. We can either commit these local changes in dev (but we are not ready to do this yet) or we can **stash** our local changes:  

```bash
git stash -u         # stash the local changes ('-u' for to stash untracked file page2)
less index.html      # note stashing has reverted local changes (index file has rolled-back to last commit and page2 is removed) 
git stash list       # note the stashed changes in stash@{0} 
git checkout master  # since local changes are stashed, we can now move between branches once again
git checkout dev     # 
git stash pop        # pop stashed changes off stash back into dev branch  
git stash list       # note stash is now empty 
git status           # note stashed changes have been restored into dev
git checkout master  # as before we CANT now switch to master again  
      error: Your local changes to the following files would be overwritten by checkout
```
To FastForward master for next demo: 
```bash
git commit -a -m "adding stashed changes"   # only when you are ready should you commit 
git checkout master  # we can now checkout master again  
git merge dev        # FF master (merge dev which is ahead into master) 
Updating 6e5753c..21b21a2
  Fast-forward
   index.html |  2 ++
   page2.html | 22 ++++++++++++++++++++++
   2 files changed, 24 insertions(+)
   create mode 100644 page2.html
```


demo8 - Reset 
--------------
* Reset: Un-commit a series of snapshots in order to re-build a new history starting from a specific snapshot
* Moves both **branchRef and HEAD** back to a specified snapshot (note, different to checkout which moves HEAD to another branch) 
* Why?  Clean-up: Allows us to rebuild a simpler branch history starting from a snapshot 
  - Can selectively choose to include changes introduced from reset snapshots by 'squashing' their changes into next/new commit   
* You should never reset snapshots that have been shared with other developers (local branches only, use revert for shared history).


* 3 Different ways to preserve (or not) the changes introduced by the 'squashed' commits when resetting:
* **`reset --soft <commitID>`** 
  - Moves **branchRef+HEAD backwards** but does not rollback Index or update workingDir 
  - Therefore, all changes from the 'squashed' commits **preserved as staged changes in the Index** (+/- local workDir changes) 
  - Running `git status` will show squashed changes in **green** as 'changes to be committed' 
  - Next commit we can selectively stage/unstage as required (+/- local workingDir changes)  
* **`reset --mixed <commitID>`** (default) 
  - Moves **branchRef+HEAD + Index** backwards but does not update workingDir
  - Therefore, index will mirror commitID and changes from 'squashed' commits are **preserved as changes in workDir**
  - Running `git status` shows only **red** files as 'changes not staged for commit'
  - Next commit will include all previously staged changes up to commitID (+/- local workDir changes) 
* **`reset --hard <commitID>`** 
  - Moves **branchRef+HEAD + Index + WorkDir** backward to mirror commitID 
  - Therefore, NO changes are preserved from 'squahsed' commits (and workDir is overwritten! - DANGEROUS) 
  - `git status` shows 'nothing to commit, working directory clean' so we need new workDir changes before we can add/commit  

```bash
git log --all --graph  --decorate --oneline  # take note of commitID when we merged branch 'footer' 
git reset <selectCommitID>             # reset to when we merged 'footer' (note all changes from later commits preseved as local changes) 
git log --all --graph  --decorate --oneline  # note (HEAD, master) are reset
	* 0336f1e (dev) adding stashed changes
	* 00e6b6f moving dev ahead of master
	*   6e5753c Merge branch 'dev'
	|\
	| * dc0cb25 modified body and paragraph in DEV
	* | 31b6416 modified in sentence and paragraph in MASTER
	|/
	* 2156bf0 updating footer to credit SCD
	* 7beb526 adding second paragraph in master
	*   dcb762b (HEAD, master) Merge branch 'footer'
	|\
	| * 3b1a4d9 adding footer
	* | b172f8d adding body content
	|/
	* d8b5ce6 fixed bug menu link
	* 0611814 removing page2, too early
	* 09cbe2f adding page2
	* 2ffc846 initial import

git status -s   # note changes from squashed commits are all preserved as local workDir changes :-) 
 M index.html  
 M page1.html
?? page2.html
git add .                                   # add local changes (if any) 
git commit -m "squashing stashed commits"   # commit all (squash all the commits occuring above reset commitID)
git log --all --graph  --decorate --oneline # note we have rebuilt our reachable history from reset point  
	* f0e2825 (HEAD, master) squashing stashed commits
	| * 116291f (dev) adding stashed changes
	| * 00e6b6f moving dev ahead of master
	| *   6e5753c Merge branch 'dev'
	| |\
	| | * dc0cb25 modified body and paragraph in DEV
	| * | 31b6416 modified in sentence and paragraph in MASTER
	| |/
	| * 2156bf0 updating footer to credit SCD
	| * 7beb526 adding second paragraph in master
	|/
	*   dcb762b Merge branch 'footer'
	|\
	| * 3b1a4d9 adding footer
	* | b172f8d adding body content
	|/
	* d8b5ce6 fixed bug menu link
	* 0611814 removing page2, too early
	* 09cbe2f adding page2
	* 2ffc846 initial import

git merge dev      # merge dev into master 
git checkout dev   # 
git merge master   # FF dev to catch up master 
git log --all --graph  --decorate --oneline
	*   24a42c0 (HEAD, master, dev) Merge branch 'dev'
	|\
	| * 116291f adding stashed changes
	| * 00e6b6f moving dev ahead of master
	| *   6e5753c Merge branch 'dev'
	| |\
	| | * dc0cb25 modified body and paragraph in DEV
	| * | 31b6416 modified in sentence and paragraph in MASTER
	| |/
	| * 2156bf0 updating footer to credit SCD
	| * 7beb526 adding second paragraph in master
	* | f0e2825 squashing stashed commits
	|/
	*   dcb762b Merge branch 'footer'
	|\
	| * 3b1a4d9 adding footer
	* | b172f8d adding body content
	|/
	* d8b5ce6 fixed bug menu link
	* 0611814 removing page2, too early
	* 09cbe2f adding page2
	* 2ffc846 initial import
```

demo9 - Revert 
---------------
* Undoes all of the changes introduced by specified commit(s) by adding a new commit that applies a set of reverse patches
* Only undoes the changes of the specific commit - It does not rollback any subsequent commits like reset does 
* E.g. a bug was introduced by a specific commit, we can revert just that commit to remove its changes
* Note, conflicts from merging the revision's reverse patch can occur if later revisions changed the same lines (this is avoided below as each reverted commit affects separate files) 

```bash
vim index.html                              
git commit -a -m "adding comment 1" 
vim page1.html                             
git commit -a -m "adding comment 2" 
vim page2.html                             
git commit -a -m "adding comment 3" 
git log --all --oneline --decorate --graph # master has moved ahead by three commits 
git show HEAD         # show changes introduced by current commit
git show HEAD~1       # show changes introduced by parent 
git show HEAD~2       # show changes introduced by grandparent 
git revert -n HEAD~2  # revert changes introduced by grandparent (-n for no commit) 
git commit -m "reverting changes" # review and apply reverted changes  
```

demo10 - Pushing/Pulling to/from a remote
----------------------------------------
* Create a remote repo on GitHub via GitHub Web portal. 
* Add our newly created remote GitHub repo to our local repo under name 'origin' 
* Push our existing repository 'master' branch to GitHub 
* Fetch, pull, remove remote

```bash
git config --list 
git remote -vv   # note no remote repos have been setup (no output) 
git branch -a    # note only local branches 
    dev
  * master
git remote add origin https://github.com/davidmeredith/<your_repo_name>.git   # copy url from github 
git remote --v   # note remote repo has been setup 
  origin  https://github.com/davidmeredith/deleteMe.git (fetch)
  origin  https://github.com/davidmeredith/deleteMe.git (push)
git branch -a    # note still only local branches even after adding remote
   dev
  * master
git branch -vv   # note no local branches are tracking remote branches yet 
git push -u origin master   (-u option to add upstream tracking reference) 
	Username for 'https://github.com': davidmeredith
	Password for 'https://davidmeredith@github.com':
	Counting objects: 55, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (55/55), done.
	Writing objects: 100% (55/55), 5.62 KiB | 0 bytes/s, done.
	Total 55 (delta 25), reused 0 (delta 0)
	To https://github.com/davidmeredith/deleteMe.git
	 * [new branch]      master -> master
	Branch master set up to track remote branch master from origin.
git branch -a    # note both local and remote branches present    
    dev
  * master
    remotes/origin/master
git branch -vv   # note local master is setup to track remote origin/master   
    dev    24a42c0 Merge branch 'dev'
  * master 51622e9 [origin/master] adding comment3
vim index.html   # make a change 
git commit -a -m "demo push" 
git push origin master    # push our master branch changes to origin 
git fetch origin master   # fetch any changes but do not merge into current branch 
git pull origin           # fetch and merge any changes into current branch 
git remote rm origin      # to locally remove remote repo
```
