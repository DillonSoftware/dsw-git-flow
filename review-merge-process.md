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

Something like this:

    $ git checkout feature/206011_fix_failing_emailservicetest
    $ git merge develop

If there are merge conflicts, you may want to contact the developer who worked the change to have them merge develop into their feature branch - they will be the most familiar with their changes, so that's probably the least risk.

***DO NOT CONTINUE UNTIL THE MERGE CONFLICTS ARE RESOLVED.***

Once there are no merge conflicts, the merged changes should be committed on your local feature branch. 

You will want to push those back up to the remote repository now:

    $ git push

Now we're ready to merge this to the `develop` branch - make sure it's still up to date:

    $ git checkout develop
    $ git pull --rebase

Now, you can see the changes between `develop` and the feature branch by doing this:

    $ git diff --name-status -R feature/206011_fix_failing_emailservicetest

If that all looks correct, you can do the merge by running this:

    $ git merge feature/206011_fix_failing_emailservicetest

Now you are ready to review the files changes to verify that the code looks right and run the tests.

If something isn't right, talk to the developer who made the change and get it corrected on the feature branch. 

***DO NOT CONTINUE UNTIL THE CODE LOOKS CORRECT, AND ALL THE TESTS ARE PASSING.***

Once everything is good, you can push the change up to the remote `develop` branch and delete the local feature branch:

    $ git push
    $ git branch -d feature/206011_fix_failing_emailservicetest

Now, the remote feature branch can be deleted as well (which is this strange looking operation):

    $ git push origin :feature/206011_fix_failing_emailservicetest

After that, the task can be marked as "code reviewed".

