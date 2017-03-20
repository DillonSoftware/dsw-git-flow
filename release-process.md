# Release process

Here's the view from 20K feet...

1. Branch per feature / task (`git flow feature start`)
2. Code review feature branches
3. Make corrections on feature branches
4. Merge to `develop` when ready (`git flow feature finish`)
5. Deploy `develop` branch nightly to DEV environment
6. Confirm functionality in DEV (dillon review and mwt preview happen here)
7. When all features are merged to `develop` and tested then deploy for final testing (`git flow release start`)
8. Deploy release snapshots to DEV for testing
9. Run final tests on the release candidate (from `master` branch) in TEST (after `git flow release finish`)
10. Any issues are treated as hotfixes on `master` branch and merged to `develop` and `master`
11. Deploy final approved candidate to production

More information about the plugin is available here: <https://bitbucket.org/atlassian/jgit-flow/wiki/goals.wiki>

> In some cases, the two are virtually identical, but the "git flow" commands are more terse (feature start, for example). In those cases, I'd probably use straight-up "git flow".

> The "git flow" tools require that each user run `git flow init` one time on their clone of the repository so that it can modify their local `.git/config` file so it knows where to put features, hotfixes, and releases. If you run this, accept the default values.

> The configuration of the jgitflow plugin lives in the pom, making it simpler to share with a team. In our case (since we use the default values), that's a minor benefit, but it is one less thing that a developer has to do.

### CI configuration

- develop - Builds the `develop` branch on each commit (what we have now). The build artifact is discarded.
- nightly deploy - Builds the `develop` branch nightly and deploys the artifact to the DEV environment. Once a release is started, the configuration is changed to build the `release` branch to the DEV environment (and TEST/PHASE2 when applicable).


### A more detailed view...

Here is a more detailed view of the process...

#### Feature branches / review process

To start a feature branch, use one of these commands:

    $ git flow feature start 123234-My_cool_task
    $ mvn jgitflow:feature-start -DfeatureName=123234-My_cool_task

> At this point the task status is: **`started`**

To facilitate the review process, feature branches would be pushed to the remote repository.

> Once the feature is code complete and ready to be reviewed by a peer developer, the task status is: **`written`**

Once the feature branch has been reviewed and corrected if necessary, it is merged back to the develop branch by running one of these commands:

    $ git flow feature finish 123234-My_cool_task
    $ mvn jgitflow:feature-finish -DfeatureName=123234-My_cool_task

> At this point the task status is: **`code reviewed`** (if you are doing a bunch of code reviews and want to hold on making these available to QA because they haven't been deployed anywhere. Once develop has been deployed to the DEV environment, you marke the task **`dillon review`**.

> NOTE: `git flow feature finish foo` WILL delete the remote branch IF you are running version `1.10.2 (AVH Edition)` of the git flow plugin. Run `git flow version` to determine your version.

> PROTIP: To delete a remote branch, use this: `git push origin :your_branch`

Functionality should be confirmed using builds from the `develop` branch in the development environment. Both DSW and MWT should do their testing in this environment, although MWT performs most/all of their testing in our TEST environment. 

> At this point the task status is: **`dillon review`**.

Once all functionality is confirmed by DS QA and MWT QA in our `DEV` environment, all tasks should be in a state of `reviewed`. At this point, a release candidate can be established.

#### Release preparation / completion

Use this command to build a stablization (release) branch:

    $ mvn clean jgitflow:release-start -DskipTests

This will prompt for the release branch name which you can typically accept the default and the next development iteration name. It will then create the release branch and switch to it. It will also bump the `develop` branch version to the next development release.

At this point you will want to change the nightly Jenkins job for this project to deploy from the release branch established, typically to all three non-production environments. This will keep things in sync and allow bug fixes to be easily deployed to all environments for all testing teams involved. Also, **update the release in the dillon task system to indicate a release candidate is built.** This indicates a release branch has been established and that Dillon QA can move tasks into **`client review`** as they see fit as the release candidate branch is being deployed nightly to all non-production environments.

MWT performs their testing in our TEST environment. If they find issues, they are marked with a **`problem`** status and corrected using the hotfix process; if the task is correct, it is marked `approved`.

Once everything is approved you'll want to submit our localization request to iBabble On _if applicable_. Wait till you get the translation files back, add them to the release branch and then deploy new builds to environment. MWT will run through any tasks that involve i18n testing.

When you are ready for the final build artifact, run the following command.

	mvn clean jgitflow:release-finish -DskipTests

This command will finish the release branch by, merging the release branch into `master`, delete the release branch; generate a tag for the release based on `master`, then build an artifact based off of `master` under the target directory; then merge `master` into `develop`. The artifact (typically war file but it can be a jar file) under the `target` directory is what you deploy to the environment. If no conflicts occur during this process you are good to go and you will run the following commands to push everything to the remote repository.

	git push --all
	git push --tags
	
IF you encounter merge conflicts during _any_ of the steps describe above when executing the `jgitflow:release-finish` command, you will need to resolve conflicts manually, merge everything manually following the steps described that the gitflow plugin follows. Following the flow is critical so be mindful of what you are merge and what you are merge into. 

#### Elastic Beanstalk Deployment

This is currently done from the AWS console. Go to the beanstalk production environment and clone the environment so you have something you _can_ swap to if things go sideways. Every production ElasticBeanstalk environment is configured for zero downtime deployments. We always have at least 2 app servers running behind the load balancer. The deployment configuration is setup so that beanstalk takes 1 app server out of the load balancer, deverts all traffic to the other app server and performs the deployment to the one it took out of the load balancer. When the healthcheck are healthy, it puts that one back into the load balancer and then performs the same steps on the other app server. It always does 1 app server at a time the way we have it configured. If things fail, the console will reflect that and you will need to jump on the server via ssh or try and get a snapshot of all the logs (in zip format) and troubleshoot what happened. Leave the clones around for a few hours; if no reported issues occur, you can terminate the cloned environment you activated as it's no longer needed.

#### Task Management

Once the release has been deployed to production, mark all tasks as **`delivered`** and set the appropriate data/time of the actual deployment on the release. You can also make sure that the Jenkins job is either only deploying develop to the DEV environment or all non-production environments if applicable. You can also disable the jenkins nightly deploy if that makes sense so it's not building and deploying what's already out there unnecessarily. 


#### Hotfix Situations

> NOTE: This is a time where REAL production issues have to be managed VERY CAREFULLY. The hotfix flow and this workflow are IDENTICAL except for the commit on `master` that they are starting from.

Once the release is deployed to production you mark all tasks **`delivered`**.

The QA team can now check the application in the production environment and verify delivery.

> At this point the task status is: `completed`

