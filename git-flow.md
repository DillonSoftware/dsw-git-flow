# Using git flow

This is a shortened version of the originals here:

- <http://nvie.com/posts/a-successful-git-branching-model/>
- <http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/>
- <http://danielkummer.github.io/git-flow-cheatsheet/>

The maven release plugin is really ideally suited for subversion, but it sucks pretty hard for git. 

A new git-flow plugin is available that does a better job here:

- <http://blogs.atlassian.com/2013/05/maven-git-flow-plugin-for-better-releases/>
- <https://bitbucket.org/atlassian/jgit-flow>

## Main branches

There are two main branches that live forever:

- `master`
- `develop`

> NOTE: If you are familiar with github flow's use of master, forget it. It's not the same.

The `origin/master` branch should alway reflect a production-ready state.

The `origin/develop` branch is where the latest delivered development changes for the next release are found.

Little active development happens on the `develop` branch - most work is done on feature branches - the `develop` branch is considered an "integration" branch where features are merged and changes are made to correct and incompatibilities between those merged features.


## Other branches

There are 3 other kinds of branches we care about.

> NOTE: There is a fourth kind of branch we don't care about - the support branch. Once we deploy 1.1 to production, 1.0 is dead to us. Support branches are created from `master` and used to support multiple concurrent versions of an application.


#### feature branches

- Only branch off from `develop`.

- Always merge back into `develop`.

- Branch naming convention: task number

- Always use `git merge --no-ff [branch-name]`.

- Always delete branch after merging it back to `develop`.

- Pushing feature branches is optional - if they are pushed, they should be rebased to the tip of the `develop` branch first (and tested and corrected if there are issues). This is still a question.

> NOTE: These are NEVER created on `master` or `release-*` branches.


#### release branches (stabilization - bug fixes in TEST)

- Only branch off from `develop`.

- Always merge back into `develop` and `master`.

- Branch naming convention: `release-*`

> NOTE: Feature branches are never created from release branches.

There should only ever be ONE release branch at a time.


#### hotfix branches (production issues)

- Always branch off from: `master`

- Always merge back into: `develop` and `master`; *maybe* merge this into a release?

- Always delete branch after merging it back to `develop` and `master`.

- Branch naming convention: `hotfix-*`

> NOTE: These are NEVER created on `develop` or `release-*` branches; only `master`.


## Our current workflow

We work primarily on `develop` (using feature branches).

Snapshots of the `develop` branch are released on-demand or nightly by Jenkins to the DEV environment.

When we think we're feature complete, we create a release branch (named `x.y`) and start deploying snapshots of that release branch to QA (test/p2). 

When we think we have a release quality build on the release branch, we create a tag (named `x.y.z`) and build a release from it.

Changes to the release branch are merged to `develop` as they are written and reviewed. 

Changes to the release branch are made either directly on the branch or in task/feature branches.

Once a tagged build has been tested, it's artifact is ready to be deployed to production.


### Thoughts/questions

Why do we branch when feature complete?


## New workflow

Still work primarily on `develop` (using feature branches). Very little actual development on `develop` - it's only for integrating feature branches and smoothing out the wrinkles introduced by merging.

Still have snapshots of the `develop` branch released on-demand or nightly by Jenkins to the DEV environment.

Still create a release branch (named `release/3.2`) and start deploying snapshots of that release branch to QA (test/p2), but do it much later - when we are ready to begin deploying to the QA environments. 

Not half-baked kinda something resembling ready-ish, but after Wagg has seen it and approved it and the mobile guys have it integrated into their apps and www is using it. 

We have feature branches so we can work on features for future release on them - we don't need to "free up" the `develop` branch for the next release. We can do bug fixes for a month there if we need to.

The key here is that a release branch should not be created when we are code complete - it should be created when we believe the code to be release quality. Release = release.

> A release branch should not have a month of development work done on top of it. If it does, we're doing it wrong. A release branch is for "dotting of i's and crossing t's", not fixing 100 bugs or adding new features.

As soon as we create the release branch, create a tag (named `3.2.0`) and build a release from it. That can be deployed to the QA environments and tested.

In the RARE event that we need to make changes to the release branch, they are merged to `develop` as they are written and reviewed.

Changes to the release branch are made directly on the branch - NEVER in task/feature branches.

Once a tagged build has been tested, it's artifact is ready to be deployed to production.


## Sample story

We're working on `hoopla-content` and it's production version is `1.2.9` (snapped the frame, there), but we're ready to start on `1.3`.

I used `git flow init` to prepare the repository and use the default names (this is a one time thing).

I'm now ready to start on task #201206 - so I do this:

    $ git flow feature start 201206-Implement_FORCE_status_for_packaging_downloads
    Switched to a new branch 'feature/201206-Implement_FORCE_status_for_packaging_downloads'

    Summary of actions:
    - A new branch 'feature/201206-Implement_FORCE_status_for_packaging_downloads' was created, based on 'develop'
    - You are now on branch 'feature/201206-Implement_FORCE_status_for_packaging_downloads'

    Now, start committing on your feature. When done, use:

         git flow feature finish 201206-Implement_FORCE_status_for_packaging_downloads

Next, mark the task as "started", and make the changes to the project and commit them to your local git repository.

> Another status would be useful here - the code is "written", but not shared yet. Maybe we use "written" and add another status of "merged"?

After they are done and tested (see the note above about what "tested" means), run this:

    $ git flow feature finish 201206-Implement_FORCE_status_for_packaging_downloads
    Switched to branch 'develop'
    Your branch is up-to-date with 'origin/develop'.
    Updating 49521a1..aa282fb
    Fast-forward
    [ --- changes listed here ---]
    Deleted branch feature/201206-Implement_FORCE_status_for_packaging_downloads (was aa282fb).

    Summary of actions:
    - The feature branch 'feature/201206-Implement_FORCE_status_for_packaging_downloads' was merged into 'develop'
    - Feature branch 'feature/201206-Implement_FORCE_status_for_packaging_downloads' has been removed
    - You are now on branch 'develop'

Now you can push your changes to origin and the task can be marked as written.

> NOTE: While the default behavior for the git-flow script is to use `--no-ff`, it WILL use a fast forward merge in the case where there is ONLY ONE commit on the branch. This is by design. The intent of using `--no-ff` is to group the related changes for a feature. If there is only one commit for that feature, they ARE grouped already (into only one commit), so creating a non-fast-forward merge is pointless.


## New workflow with gitflow maven plugin

This flow starts when you want to create a release branch (ie. release/1.0.0). 

	$ git checkout develop
	$ git pull
	$ mvn clean jgitflow:release-start

This plugin is interactive and will prompt you with two questions: 

What is the release version for "<PROJECT NAME>"? 1.0.0
What is the development version for "<PROJECT NAME>"? 1.1.0

A release branch is expected to be very short lived, so we give it a micro release and any bug fixes that need to occur for this release (which should be few and far between b/c our snapshots should have exhausted the real QA process, will be done as a hotpatch). 

Push the release branch to the remote server

	$ git push
	
Now it's time to snap a release from the release branch to be deployed to the server.

	$ mvn clean package -DskipTests
	
Upload the resulting artifact (typically either a war or zip file) to either beanstalk or camel server.

...

As indicated above, there shouldn't be much activity on this release branch, if a bug fix is needed, a hotfix will be used (more to come on this).

Once the release is truly blessed, you finish the release with the following command:

	$ mvn clean jgitflow:release-finish
	
This will perform the necessary merges to develop/master and produce the resulting artifact that you wish to deploy. Keep in mind that all of the changes made are to your _local repository only_. You must push your develop and master branch to the remote when you are finish.

## Questions / Tips

  - how do we do prototypes quickly? 
    - on-demand beanstalk environments? is the value more than the cost?
    - skype?
  - how do we do a unified build with everything?
    - the `develop` branch is that unified view
  - what do we do for features that require other features?
    - avoid this scenario if possible :)
    - do them in a linear fashion instead of in parallel
    - decompose them into 3 features (shared, feature A, feature B)
    - merge the features into another (non-develop) branch?


### Remember git is not subversion

Work the task - once you're done, commit it and run the tests. 

In that order. 

Really.

If you get test failures, that's OK - fix them up and then amend your commit by adding the changes, then using `git commit --amend`, or if the changes are more significant, create a new commit. Then run the tests again on a fully committed repository.

Always run the test suite on a fully committed repository, because if you do, then you know that what you push is what you tested.


### Keep your feature branch up-to-date

You can merge the `develop` branch into your feature branch. It's a reasonable way to keep your feature current with ongoing development.

### Do we push our feature branches?

Generally, yes. Doing so make code reviews easier.

If everyone pushes their feature branches, it's also easier to integrate changes in someone else's branch into your feature (if needed).

### How do I get tags created by others?

If you only need to get them once in a while, you can do this:

    $ git pull --tags

If you want to ALWAYS fetch them, you can add this to the `.git/config` for your cloned repository (the first part is likely there already):

    [remote "origin"]
        url = git@dillonsoftware.git.beanstalkapp.com:/dillonsoftware/hoopla-content.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        fetch = +refs/tags/*:refs/tags/*

