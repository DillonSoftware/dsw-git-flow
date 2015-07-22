# Review / Merge process

> NOTE #1: Remember that git is NOT subversion.

> NOTE #2: Be sure you have a recent git release (I'd suggest 2.3+).

> NOTE #3: This is not a git tutorial, it is a process explanation - if you see a git command and think "What's that mean?", you should google it.

#### Get the latest `develop` branch from the remote repository:

Always start with the latest revision of the remote `develop` branch:

    $ git checkout develop
    $ git pull --rebase

#### Checkout the feature branch to be reviewed / merged

You need to checkout the feature branch and merge in the latest changes from the `develop` branch and push them back up to the remote repository. 

Something like this from a terminal

    $ git checkout feature/206011_fix_failing_emailservicetest
    
OR

	Using IDEA, find the remote feature branch in the __Git:{branch}__ menu/label in the lower right corner of the window.  Check it out as a local branch. 


Now merge the latest code from the develop branch into the current feature branch you have checked out

	$ git merge develop


If there are merge conflicts, you may want to contact the developer who worked the change to have them merge develop into their feature branch - they will be the most familiar with their changes, so that's probably the least risk.

***DO NOT CONTINUE UNTIL THE MERGE CONFLICTS ARE RESOLVED.***

If there were any sql or java changes, run the full suite of tests now on the feature branch. If there are any problems or a coverage drop, you can either fix them yourself if they are easy, or throw it back to the developer.

Finally, the code review part of "code review". Look at the diff of each changed file while on the feature branch from within IDEA. Check it for

- compliance with established formatting, naming, and other conventions 
- sound, well organized code 
- (optional) that it actually works

If it fails your review, mark the task problem and leave the developer a note as to what the problem is.

If all is good, then it's time to finish up the feature branch.  You can do this with SourceTree or you can just use the terminal:

	$ git flow feature finish {branch name}  <-- no need for feature/ path

Finishing the feature branch merges the feature branch back into develop, deletes the _local_ feature branch, and checks out develop.

Make sure your develop branch is still up to date:

    $ git pull --rebase
	
Run full suite of unit tests on develop to make sure there are no issues or coverage drops.	
If something isn't right, talk to the developer who made the change and get it corrected on the feature branch. 

***DO NOT CONTINUE UNTIL THE CODE LOOKS CORRECT, AND ALL THE TESTS ARE PASSING.***

If the tests all pass and everything is good, you now need to push your local develop branch to the remote repository:

    $ git push
   

Delete the remote feature branch.  The `git flow feature finish` you did earlier only deleted the local feature branch.  You can delete the remote feature branch either with SourceTree or in terminal:

	$ git push origin :feature/206011_fix_failing_emailservicetest

If you use SourceTree, don't "force delete" - we want git to make sure the remote branch has been merged; "force delete" omits that check.

__Change the task status to "code reviewed"__.

#### Your code review is complete
