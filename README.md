Objective-C/Swift Framework Deployment made easy
================

Note this is still a **work in progress**.

Creating a new release for an Objective-C/Swift open source framework is a tedious process. This fastlane setup is an attempt to streamline that process by creating a single button to generate a new release, allowing developers to focus on great updates, without the overhead of creating new releases.

It is recommended to have a high level understanding of _what_ fastlane is, but it is not required. You can read up more on fastlane [here](https://github.com/fastlane/fastlane/tree/master/docs).

# Installation
To install fastlane, simply use gem (related: [Should I use sudo?](http://stackoverflow.com/a/2119413)):

```
[sudo] gem install fastlane
```

# Why?
The goal is to create a one button release like so with almost no overhead for a developer. 

```
fastlane prepare_framework_release version:3.0.0 env --deploy
```

That does the following:

 1. Verifies the git branch is clean
 * Ensures the lane is running on the master branch
 * Verifies the Github milestone is ready for release (no open issues, and at least one closed issue)
 * Pulls the remote to verify the latest the branch is up to date
 * Updates the version of the info plist used by the framework
 * Updates the version of the podspec
 * Generates a [changelog](https://github.com/AFNetworking/AFNetworking/blob/master/CHANGELOG.md#262-11062015) based on the Github [milestone](https://github.com/AFNetworking/AFNetworking/issues?q=milestone%3A2.6.2+is%3Aclosed)
 * Updates the changelog file
 * Commits the changes to master
 * Pushes the commited branch to the remote
 * Creates a new tag
 * Pushes the tag to the remote
 * Runs the tests on the CI server

When the tests have passed on CI, the `complete_framework_release` lane will _automatically_ run:
 
 1. Generates a changelog for the Github Release Page
 * Creates a [Github Release](https://github.com/AFNetworking/AFNetworking/releases/tag/2.6.2)
 * Builds Carthage Frameworks
 * Uploads Carthage Framework to Github Release
 * Pushes podspec to pod trunk
 * Lints the pod spec to ensure it is valid
 * Closes the associated Github milestone


## Github Management Best Practices
In order to get the _most_ out of this tool, it is recommended you manage your Github repo with the following best practices. It's easy to do, and gives you a lot of automation power with fastlane. Below is how AFNetworking is being managed:

* Any changes that should be referenced in the changelog shoud be **merged using pull requests.**
* **The title of the pull request should be consice**, and ready to display in the changelog. This means as the admin of the repo, it may be appropriate for you to edit a pull request's title to make it more meaningful.
* **All releases should be managed with a Github milestone**. Not only does this provide a [well documented](https://github.com/AFNetworking/AFNetworking/issues?q=milestone%3A2.6.2) place for users to get a clear view of what went in to a release, but it will also serve as the main component for changelog generation. The milestone name should match the planned tag name.
* **All issues merged should be grouped into one of five categories** using Github labels. This allows the changelog to be divided into multiple, logical sections, like [so](https://github.com/AFNetworking/AFNetworking/blob/master/CHANGELOG.md#262-11062015). You can see an example of an AFNetworking release managed this [way](https://github.com/AFNetworking/AFNetworking/issues?q=milestone%3A2.6.2+is%3Aclosed).
	* Added
	* Updated
	* Changed
	* Fixed
	* Removed

## Getting Started
Getting up and running is a simple process.

### Local Fastlane
* Create a `fastlane` directory in your repo
* In the `fastlane` directory, create an empty `Fastfile`
* Import the AFNetworking fastlane setup by adding the following to the top of the file:

```
import_from_git(
  url: 'git@github.com:AFNetworking/fastlane.git', 
  branch: '0.0.1'
)
```

Click [here](https://github.com/fastlane/fastlane/blob/master/docs/Actions.md#import_from_git) for more information about importing fastlanes.

### Environment Variables
[Environment variables](https://github.com/fastlane/fastlane/blob/master/docs/Advanced.md#environment-variables) are an easy and powerful way to manage how fastlane builds your project. Fastlane searches your `fastlane` directory, and loads environment variables in the following order:

* `.env`: Values for your project that typically never change
* `.env.default`: Sensible defaults for values in your project that may change depending on the environment
* Finally, a custom environment file passed in to the the fastlane: Specific variables for the environment you want to test.

Without any additional parameters, fastlane will automatically load `.env` and `.env.default`. You can use the `--env` flag to pass in an additional environment variables file.

These files are meant to be checked in to your repo in the `fastlane` directory, and should define everything except sensitive values (like passwords and API tokens).

Here is an example of the AFNetworking deploy environment variable file:

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

For more configuration options, please the lane documentation below.

**TODO** Link to AFNetworking Setup.

### CI

Configuring your repository for continuous integration allows your project to constantly be under test, and gives you greater confidence in your release. Both [Travis CI](http://docs.travis-ci.com/user/getting-started/) and [Circle CI](https://circleci.com/docs/getting-started) offer free plans for open source projects. This guide will focus on Travis CI integration.

The AFNetworking `.travis.yml` file is setup as follows:

```
language: objective-c
osx_image: xcode7.1
sudo: false
env:
  global:
  - LC_CTYPE=en_US.UTF-8
  - LANG=en_US.UTF-8
  - FASTLANE_LANE=ci_commit
  matrix:
    - FASTLANE_ENV=ios81
    - FASTLANE_ENV=ios82
    - FASTLANE_ENV=ios83
    - FASTLANE_ENV=ios84
    - FASTLANE_ENV=ios90
    - FASTLANE_ENV=ios91
    - FASTLANE_ENV=osx
    - FASTLANE_ENV=tvos90
before_install:
  - gem install fastlane --no-rdoc --no-ri --no-document --quiet
  - gem install cocoapods --no-rdoc --no-ri --no-document --quiet
  - gem install xcpretty --no-rdoc --no-ri --no-document --quiet
script:
  - set -o pipefail
  - fastlane $FASTLANE_LANE configuration:Debug --env $FASTLANE_ENV
  - fastlane $FASTLANE_LANE configuration:Release --env $FASTLANE_ENV
deploy:
  provider: script
  script: fastlane complete_framework_release --env deploy
  on:
    tags: true
```

This can be read as follows:

* Declare the project is Objective-C, should run on the Xcode 7.1 machine, and should not use `sudo`
* Set the following global environment variables for all runs: `LC_CTYPE`, `LANG`, and `FASTLANE_LANE`
	* By setting `FASTLANE_LANE`, branches can easily be configured to run a different lane if needed.
* Using matrix, spin up multiple concurrent Travis jobs, and run them with the specific environment variables for that line. 
	* In this case, environment variables are being declared for the fastlane environment variable file name. These represent iOS 8.0-9.1, OS X, and tvOS 9.0. I can _easily_ add additional test targets as new versions are released by add a new environment variable file, and updating the `.travis.yml` to include it in the matrix.
* Before anything runs, install fastlane, cocoapods, and xcpretty.
* Run the following script for every job.
	* This runs the lane provided by `$FASTLANE_LANE`, which in this case is `ci_commit`. The lane is run twice, once in Debug and once in Release, confirming both the tests and example project compile and pass all tests in both configurations.
* To deploy, run the `complete_framework_release` lane with the deploy environment, and only do it on a tag. Configure additional options here if needed.
	* Secure environment variables can be configured using the [Travis web interface](http://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings). Any variable marked as hidden is only available to Travis jobs that we're initiated as the result of a trusted source, meaning anyone who submits a pull request to the repo without push access won't have access to these variables when the job runs on CI, keeping your passwords secure. 

### Github API Token

To create, edit, and close Github Milestones and Releases, and to access private repositories, the environment variable `$GITHUB_API_TOKEN` must be provided using a secure environment variable, as described above. **DO NOT** check this into a public repository.

### Pod Trunk API Token

Kyle Fuller has an excellent [write up](https://fuller.li/posts/automated-cocoapods-releases-with-ci/) on how to generate Cocoapods trunk access token. Follow that guide to setup a secure environment variable for `$COCOAPODS_TRUNK_TOKEN`.


# Available Fastlane Lanes
The lanes provided are divided into two groups: lanes meant for for testing, and lanes meant for deployment.

---

## Test Lanes
### ci_commit
Runs tests and builds example for the given environment

The lane to run by ci on every commit. This lanes calls the lanes `test_framework` and `build_example`.

####Example:

```
fastlane ci_commit configuration:Debug --env ios91
```

####Options

 * **`configuration`**: The build configuration to use. (`AF_CONFIGURATION`)


### test_framework
Runs all tests for the given environment

Set `scan` action environment variables to control test configuration

####Example:

```
fastlane test_framework configuration:Debug --env ios91
```

####Options

 * **`configuration`**: The build configuration to use.


### build_example
Builds the example file

Set [`xcodebuild`](https://github.com/fastlane/fastlane/blob/master/docs/Actions.md#xcodebuild) action environment variables to control build configuration

####Example:

```
fastlane build_example configuration:Debug --env ios91
```

####Options

 * **`configuration`**: The build configuration to use.

---
## Deployment Lanes
To deploy a new version of AFNetworking, a contributor does the following:

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

### prepare_framework_release

Prepares the framework for release

This lane should be run from your local machine, and will push a tag to the remote when finished.

####Actions Performed
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

####Example:

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

#####Environment Variable Only Options
 * **`GITHUB_OWNER`**: The owner of the Github repository, used in changelog generation and Github release management
 * **`GITHUB_REPOSITORY`**: The Github repository, used in changelog generation and Github release management
 * **`GITHUB_API_TOKEN`**: The Github API token, used in changelog generation and Github release management. It is recommended to provide this securely through a CI service. Generate one at [https://github.com/settings/tokens](https://github.com/settings/tokens)

### complete_framework_release

Completes the framework release

This lane should be from a CI machine, after the tests have passed on the tag build.

####Actions Performed
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

####Example:

```
fastlane complete_framework_release --env deploy
```

####Options
#####CLI Options
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

#####Environment Variable Only Options
 * **`GITHUB_OWNER`**: The owner of the Github repository, used in changelog generation and Github release management
 * **`GITHUB_REPOSITORY`**: The Github repository, used in changelog generation and Github release management
 * **`GITHUB_API_TOKEN`**: The Github API token, used in changelog generation and Github release management. It is recommended to provide this securely through a CI service. Generate one at [https://github.com/settings/tokens](https://github.com/settings/tokens)
 * **`CARTHAGE_FRAMEWORK_NAME`**: The name of the generated Carthage framework
