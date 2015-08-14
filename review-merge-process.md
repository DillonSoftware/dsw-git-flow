# Review / Merge process

> NOTE #1: Remember that git is NOT subversion.

> NOTE #2: Be sure you have a recent git release (I'd suggest 2.3+, the latest version as of today is 2.5.0).

> NOTE #3: This is not a git tutorial, it is a process explanation - if you see a git command and think "What's that mean?", you should google it.

__Change the task status to "in review" now__.

#### Make sure the task has all the required information

Does it have test notes so someone can verify that it works as expected?

> NOTE: If you are reviewing a task that you are familiar with, and think "Gee, I don't need testing notes, I know how this should work...", you're wrong. :) Add testing notes to the task or ask the developer to do so. The task system is the system of record for tasks, not your brain. If you get hit by a truck, someone else should be able to review and verify that the code works.

#### Get the latest `develop` branch from the remote repository:

Always start with the latest revision of the remote `develop` branch:


    $ git checkout develop
    $ git pull --rebase


#### Checkout the feature branch to be reviewed / merged

You need to checkout the feature branch and merge in the latest changes from the `develop` branch and push them back up to the remote repository. 

Something like this from a terminal


    $ git checkout feature/206011_fix_failing_emailservicetest


...OR using IDEA, find the remote feature branch in the __Git:{branch}__ menu/label in the lower right corner of the window.  Check it out as a local branch. 

Now merge the latest code from the develop branch into the current feature branch you have checked out


    $ git merge develop


If there are merge conflicts, you may want to contact the developer who worked the change to have them merge develop into their feature branch - they will be the most familiar with their changes, so that's probably the least risk.

***DO NOT CONTINUE UNTIL THE MERGE CONFLICTS ARE RESOLVED.***

If there were any sql or java changes, run the full suite of tests now on the feature branch. If there are any problems or a coverage drop, you can either fix them yourself if they are easy, or throw it back to the developer.

Finally, the code review part of "code review". Look at the diff of each changed file while on the feature branch from within IDEA. 

At a minimum, check for:


 - readability: compliance with established conventions
   - formatting, naming, logging, comments, and spelling
   - whitespaceisfreesoyoushoulduseitbecauseitmakesthingseasiertoread
 - structure: sound, well-organized code
   - no Rube Goldberg machines!
   - use checkstyle and idea's analysis (at the very least)
   - is the structure architecturally thought out or did it evolve organically by trial-n-error?
 - function: does it actually work?
   - use the test notes on the task to verify that the change works as expected
   - run the full unit test suite before AND after merging to develop


Note that the last thing on the list is "Does it work?". That's intentional: "It works." is not the same as "It's correct." for this team. Making something "work" is no excuse for sloppy hackery.

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

__Set the release field's value on the task to the version in the pom.__

__Change the task status to "code reviewed".__

#### Your code review is complete


