An introduction to Git via demos 
=================================
Tip: To view this markdown file (.md) as a webpage, install a markdown viewer/plugin into your browser.

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
git diff                   # show modified lines between workDir/Index
git add .
git diff --staged          # show modified lines between index/lastCommit
git commit -m "first modifications" 
git status                 # note clean working dir
git log --oneline --graph --decorate --all # note HEAD-pointsTo->master-pointsTo->hash 
git show <commitID>        # to show what was changed in commit
```



demo4 - Three Way Merge then FastForward 
-----------------------------------------
```bash
git log --oneline --graph --decorate --all  # note fork start position (footer in footer and text in master)
	* 568438c (HEAD, master) adding paragraph
	| * 33991ba (footer) adding footer
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git show HEAD          # show what was added in last commit
git checkout footer
git show HEAD          # note un-commenting of footer
less index.html        # note footer is *not* in comments and no paragraph
git checkout master
less index.html        # note footer is in comments and added paragraph (line 15)
git merge footer       # merge footer into master 
	Auto-merging index.html
	Merge made by the 'recursive' strategy.
	 index.html | 4 ++--
	 page1.html | 4 ++--
	 2 files changed, 4 insertions(+), 4 deletions(-)

less index.html      # note footer and paragraph both present
git log --oneline --graph --decorate --all      # note master now ahead of footer
	*   5e88ea0 (HEAD, master) Merge branch 'footer'
	|\
	| * 33991ba (footer) adding footer
	* | 568438c adding paragraph
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git checkout footer     # bring footer up to date, first checkout footer 
git merge master        # merge master into footer 
	Updating 33991ba..5e88ea0
	Fast-forward
	 index.html | 2 +-
	 1 file changed, 1 insertion(+), 1 deletion(-)

git log --oneline --graph --decorate --all  # note both branches up to date
	*   5e88ea0 (HEAD, master, footer) Merge branch 'footer'
	|\
	| * 33991ba adding footer
	* | 568438c adding paragraph
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit
 

```

demo6 - Merge Conflict
-----------------------

```bash
git log --oneline --graph --decorate --all  # note fork start position (footer in footer and text in master)
	* 568438c (HEAD, master) adding paragraph
	| * 33991ba (footer) adding footer
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit

git show HEAD           # note paragraph text added
less index.html         # note footer is in comments
vim index.html          # add 'Version 1 body text' below <div>index body</div>
git commit -a -m "adding v1 body text"
git checkout footer
less index.html         # note footer is not in comments 
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
```bash
git branch
	* dev
	  master

git log --oneline --graph --decorate --all
	* 798601c (HEAD, dev) moving dev ahead of master
	*   9c14db5 (master) Merge branch 'footer'
	|\
	| * 33991ba adding footer
	* | 568438c adding paragraph
	|/
	* 7af7c7a removing page2 - too soon
	* 3a3b05f adding page2
	* 43b9178 initial commit
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






