Objective-C/Swift Framework Deployment made easy
================

Note this is still a **work in progress**.

Creating a new release for an Objective-C/Swift open source framework is a tedious process. This fastlane setup is an attempt to streamline that process by creating a single button to generate a new release, allowing developers to focus on great updates, without the overhead of creating new releases.

It is recommended to have a high level understanding of _what_ fastlane is, but it is not required. You can read up more on fastlane [here](https://github.com/fastlane/fastlane/tree/master/docs).

## Installation
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


# Github Management Best Practices

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

# Getting Started
There are primary use cases for these fastlanes: testing and deployment. When used together, a full continuous integration system can be created.

## Testing
Getting the testing harness configured is an easy process.

1. Configure the scheme's test action to run a test target, and confirm all tests pass locally
2. Setup the Fastfile
2. Setup the env files
3. Setup CI (optional)
4. Profit 💰

Follow the [getting started guide](docs/TestingGuide.md) to configure the framework for testing.

## Deployment

Getting up and running is a simple process:

1. Setup the Fastfile
2. Setup the env files
3. Setup CI (optional)
4. Generate a Github API Token
5. Generate a Pod Trunk Access Token
6. Profit 💰

A full deployment [guide](docs/DeploymentGuide.md) has been created to help get developers get started with automating deployment.

# Available Fastlane Lanes
The lanes provided are divided into two groups: lanes meant for for testing, and lanes meant for deployment. In combination with the Github best practices above, developers can fully automate the release process.

* [Test Lanes](docs/TestLanes.md)
* [Deployment Lanes](docs/DeployLanes.md)
