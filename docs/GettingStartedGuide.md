# Getting Started
Getting up and running is a simple process, which is explained in detail below:

1. Setup the Fastfile
2. Setup the env files
3. Setup CI (optional)
4. Generate a Github API Token
5. Generate a Pod Trunk Access Token

## Local Fastlane
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

## Environment Variables
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

For more configuration options and environment variables, please the [deployment lane documentation](DeployLanes.md).

**TODO** Link to AFNetworking Setup.

## CI

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

### Deploying without CI
It is possible to deploy without a CI setup. The `complete_framework_release` lane can be run locally if `skip_ci_check:true` is passed a parameter when run.

## Github API Token

To create, edit, and close Github Milestones and Releases, and to access private repositories, the environment variable `$GITHUB_API_TOKEN` must be provided using a secure environment variable, as described above. **DO NOT** check this into a public repository.

## Pod Trunk API Token

Kyle Fuller has an excellent [write up](https://fuller.li/posts/automated-cocoapods-releases-with-ci/) on how to generate Cocoapods trunk access token. Follow that guide to setup a secure environment variable for `$COCOAPODS_TRUNK_TOKEN`.