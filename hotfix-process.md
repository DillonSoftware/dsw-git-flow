# Hotfix process

Hot fixes should be very rare - following the release process, the releases should be pretty stable. Problems do have a way of sneaking through, so here's how we deal with them.

Create a hotfix branch using one of these two commands:

    $ git flow hotfix start 2.0.4
    $ mvn jgitflow:hotfix-start -DreleaseVersion=2.0.4

> At this point the task status is: `started`

Both of those branch from `master` to `hotfixes/2.0.4`

**You should immediately change the version number in the pom.xml to represent the hotfix version.**

Changes are made directly to the hotfix branch, not a branch from it. These changes are pushed to the remote repository when completed.

> At this point the task status is: `written`

Snapshots of the hotfix branch should be deployed to the DEV environment for verification. 

> At this point the task status is either: `dillon review` or `client review`

Once they are confirmed, the hotfix should be finished.

> At this point the task status is: `reviewed`

To finish a hotfix, use one of these two commands:

    $ git flow hotfix finish 2.0.4
    $ mvn jgitflow:hotfix-finish -DreleaseVersion=2.0.4

Both of those will merge the hotfix from the branch to both the `master` and `develop` branches.

The second command will also build the artifact for deployment to TEST for final testing and adjust the pom to indicate that it's a real release and not a snapshot.

> At this point the task status is: `testing`

Now the final testing is done.

You now need to push the changes that were merged into your local develop and master branches up to the remote.

	$ git checkout develop
	$ git status #should show that you are 'ahead' and need to push
	$ git push
	$ git checkout master
	$ git status #should show that you are 'ahead' and need to push
	$ git push
	
**Lastly, you need to delete the remote hotfix branch that was created when you started the hotfix. The easiest way to do this is via SourceTree.**

> At this point the task status is either `approved` or `problem`

If there is still a problem, we start over (create a new hotfix branch, etc...).

If the build is approved (the task status value is `approved`), the artifact can be moved to production.

