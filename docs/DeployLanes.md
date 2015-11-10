# Deployment Lanes
The deployment lanes _significantly_ cut down on the time required to release a new version of a framework, taking a process that could take an hour or more down to seconds. Developers simply need to run the following command:

```
fastlane prepare_framework_release version:{THE NEW VERSION} --env deploy
```
And profit💰

What actually happens is the following:

 1. A contributer will run `fastlane prepare_framework_release version:{THE NEW VERSION} --env deploy`. See the documentation below for all actions that are performed by `prepare_framework_release`
 2. On success, the tag will automatically be pushed to the remote
 3. Travis will generate a new build job
 4. On a successful build from a tag, travis will run `fastlane complete_framework_release --env deploy`, which publishes all released artifacts. See the documentation below for all actions that are performed by `complete_framework_release`


It is recommended to manage the deployment lanes with a .env file, such as the following:

```
DEPLOY_BRANCH=master
DEPLOY_PLIST_PATH=Framework/Info.plist
DEPLOY_PODSPEC=AFNetworking.podspec
DEPLOY_REMOTE=origin

DEPLOY_CHANGELOG_PATH=CHANGELOG.md
DEPLOY_CHANGELOG_DELIMITER=---

# Used for CHANGELOG Generation and Github Release Management
GITHUB_OWNER=AFNetworking
GITHUB_REPOSITORY=AFNetworking
# CI Should Provide GITHUB_API_TOKEN

CARTHAGE_FRAMEWORK_NAME=AFNetworking
```

## prepare_framework_release

Prepares the framework for release

This lane should be run from your local machine, and will push a tag to the remote when finished.

###Actions Performed
 * Verifies the git branch is clean
 * Ensures the lane is running on the master branch
 * Verifies the Github milestone is ready for release
 * Pulls the remote to verify the latest the branch is up to date
 * Updates the version of the info plist used by the framework
 * Updates the version of the podspec
 * Generates a changelog based on the Github milestone
 * Updates the changelog file
 * Commits the changes
 * Pushes the commited branch
 * Creates a tag
 * Pushes the tag

###Example:

```
fastlane prepare_framework_release version:3.0.0 --env deploy
```

####Options

It is recommended to manage these options through a .env file. See `fastlane/.env.deploy` for an example.

#####CLI Options
 * **`version`** (required): The new version of the framework
 * **`allow_dirty_branch`**: Allows the git branch to be dirty before continuing. Defaults to false
 * **`remote`**: The name of the git remote. Defaults to `origin`. (`DEPLOY_REMOTE`)
 * **`allow_branch`**: The name of the branch to build from. Defaults to `master`. (`DEPLOY_BRANCH`)
 * **`skip_validate_github_milestone`**: Skips validating a Github milestone. Defaults to false
 * **`skip_git_pull`**: Skips pulling the git remote. Defaults to false
 * **`skip_plist_update`**: Skips updating the version of the info plist. Defaults to false
 * **`plist_path`**: The path of the plist file to update. (`DEPLOY_PLIST_PATH`)
 * **`skip_podspec_update`**: Skips updating the version of the podspec. Defaults to false
 * **`podspec`**: The path of the podspec file to update. (`DEPLOY_PODSPEC`)
 * **`skip_changelog`**: Skip generating a changelog. Defaults to false.
 * **`changelog_path`**: The path to the changelog file. (`DEPLOY_CHANGELOG_PATH`)
 * **`changelog_insert_delimiter`**: The delimiter to insert the changelog after. (`DEPLOY_CHANGELOG_DELIMITER`)

####Environment Variable Only Options
 * **`GITHUB_OWNER`**: The owner of the Github repository, used in changelog generation and Github release management
 * **`GITHUB_REPOSITORY`**: The Github repository, used in changelog generation and Github release management
 * **`GITHUB_API_TOKEN`**: The Github API token, used in changelog generation and Github release management. It is recommended to provide this securely through a CI service. Generate one at [https://github.com/settings/tokens](https://github.com/settings/tokens)

## complete_framework_release

Completes the framework release

This lane should be from a CI machine, after the tests have passed on the tag build.

###Actions Performed
 * Verifies the lane is running on a CI machine
 * Verifies the git branch is clean
 * Ensures the lane is running on the master branch
 * Pulls the remote to verify the latest the branch is up to date
 * Generates a changelog for the Github Release
 * Creates a Github Release
 * Builds Carthage Frameworks
 * Uploads Carthage Framework to Github Release
 * Pushes podspec to pod trunk
 * Lints the pod spec to ensure it is valid
 * Closes the associated Github milestone

###Example:

```
fastlane complete_framework_release --env deploy
```

###Options
####CLI Options
It is recommended to manage these options through a .env file. See `fastlane/.env.deploy` for an example.

 * **`skip_ci_check`**: Bypass the requirement to run on a CI machine. Defaults to false
 * **`version`**: The new version of the framework. Defaults to the last tag in the repo
 * **`allow_dirty_branch`**: Allows the git branch to be dirty before continuing. Defaults to false
 * **`remote`**: The name of the git remote. Defaults to `origin`. (`DEPLOY_REMOTE`)
 * **`allow_branch`**: The name of the branch to build from. Defaults to `master`. (`DEPLOY_BRANCH`)
 * **`skip_github_release`**: Skips creating a Github release. Defaults to false
 * **`skip_carthage_framework`**: Skips creating a carthage framework. If building a swift framework, this should be disabled. Defaults to false.
 * **`skip_pod_push`**: Skips pushing the podspec to trunk.
 * **`skip_podspec_update`**: Skips updating the version of the podspec. Defaults to false
* **`skip_closing_github_milestone`**: Skips closing the associated Github milestone. Defaults to false

####Environment Variable Only Options
 * **`GITHUB_OWNER`**: The owner of the Github repository, used in changelog generation and Github release management
 * **`GITHUB_REPOSITORY`**: The Github repository, used in changelog generation and Github release management
 * **`GITHUB_API_TOKEN`**: The Github API token, used in changelog generation and Github release management. It is recommended to provide this securely through a CI service. Generate one at [https://github.com/settings/tokens](https://github.com/settings/tokens)
 * **`CARTHAGE_FRAMEWORK_NAME`**: The name of the generated Carthage framework
