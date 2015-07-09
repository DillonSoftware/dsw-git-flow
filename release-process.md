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

> NOTE: I think the jgitflow plugin may be the way to go for us. The "git flow" scripts are not bad, but the jgitflow plugin is geared specifically for maven-based projects using git flow and automate much of the monkey work. More information about the plugin is available here: <https://bitbucket.org/atlassian/jgit-flow/wiki/goals.wiki>

> In some cases, the two are virtually identical, but the "git flow" commands are more terse (feature start, for example). In those cases, I'd probably use straight-up "git flow".

> The "git flow" tools require that each user run `git flow init` one time on their clone of the repository so that it can modify their local `.git/config` file so it knows where to put features, hotfixes, and releases. If you run this, accept the default values.

> The configuration of the jgitflow plugin lives in the pom, making it simpler to share with a team. In our case (since we use the default values), that's a minor benefit, but it is one less thing that a developer has to do.

### CI configuration

- develop - Builds the `develop` branch on each commit (what we have now). The build artifact is discarded.
- nightly deploy - Builds the `develop` branch nightly and deploys the artifact to the DEV environment. Once a release is started, the configuration is changed to build the `release` branch to the DEV environment.
- test deploy - Builds the `master` branch on each commit and deploys to the TEST environment. This will have the side-effect of making the build available to deploy to the P2 and PROD environments.

### A more detailed view...

Here is a more detailed view of the process...

#### Feature branches / review process

To start a feature branch, use one of these commands:

    $ git flow feature start 123234-My_cool_task
    $ mvn jgitflow:feature-start -DfeatureName=123234-My_cool_task

> At this point the task status is: `started`

To facilitate the review process, feature branches would be pushed to the remote repository.

> Once the feature is ready to be merged, the task status is: `written`

Once the feature branch has been reviewed and corrected if necessary, it is merged back to the develop branch by running one of these commands:

    $ git flow feature finish 123234-My_cool_task
    $ mvn jgitflow:feature-finish -DfeatureName=123234-My_cool_task

> At this point the task status is: **`merged`** (NEW STATUS)

> QUESTION: Do the `git flow` tools remove the remote branches on completion? If not, we'll want to do that frequently or we'll have a mess on our hands.

> NOTE: `git flow feature finish foo` does NOT delete the remote branch. Bummer.

> PROTIP: To delete a remote branch, use this: `git push origin :your_branch`

Functionality should be confirmed using builds from the `develop` branch in the development environment. Both DSW and MWT should do their testing in this environment.

> At this point the task status is: `dillon review`

Once all functionality is confirmed by both DS QA and MWT QA in our `DEV` environment, all tasks should be in a state of `reviewed`. At this point, a release candidate can be established.

#### Release preparation / completion

Use this command to release the codebase and build an artifact to deploy:

    $ mvn clean jgitflow:release-start jgitflow:release-finish -DskipTests

This will prompt for the release name and the next development iteration name. It will then create the release branch and switch to it, build an artifact under the target directory, merge the changes from the release branch to the develop branch and switch you back to the develop branch. The artifact (typically war file) under the `target` directory is what you deploy to the environment. Also, it deletes the release branch that was created locally by the `jgitflow:release-start` plugin.
That second command will create the build artifact as well as merge the release branch to the `master` and `develop` branches.

> At this point the task status is: `reviewed`

This is where MWT does their final testing. If they find issues, they are marked with a `problem` status and corrected using the hotfix process; if the task is correct, it is marked `approved`.

> NOTE: This is a time where REAL production issues have to be managed VERY CAREFULLY. The hotfix flow and this workflow are IDENTICAL except for the commit on `master` that they are starting from.

Once the release is ready to go to production (all tasks are `approved`), the artifact can be deployed to the production environment.

> At this point the task status is: `delivered`

The QA team can now check the application in the production environment and verify delivery.

> At this point the task status is: `completed`

