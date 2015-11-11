# Getting Started with Testing

Getting the testing harness configured is an easy process. 

1. Configure the scheme's test action to run a test target, and confirm all tests pass locally
2. Setup the Fastfile
2. Setup the env files
3. Setup CI (optional)

The end goal is to easily be able to run the tests with the following command:

```
fastlane test_framework --env ios91
```

AFNetworking provides three fastlanes designed for testing

* `test_framework`
* `build_example`
* `ci_commit`, which simply rolls both of the above lanes into one.

You can view additional documentation about these lanes [here](TestLanes.md).

## Local Fastfile

1. Create a `fastlane` directory in your repo
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

The AFNetworking fastlanes the following builtin fastlane actions:

* **[scan](https://github.com/fastlane/scan)**: The fastlane action to run Xcode tests. Run `fastlane action scan` to see all available environment variables.
* **[xcodebuild](https://github.com/fastlane/fastlane/blob/master/docs/Actions.md#xcodebuild)**: A fastlane wrapper around Xcode build to build the project example. Run `fastlane action xcodebuild` to see all available environment variables.

Here is how AFNetworking configures the environment variables.

### .env
The `.env` file is loaded first by fastlane, and should be used to defined static variables that you do not expect to change.

```
AF_WORKSPACE="AFNetworking.xcworkspace"

AF_IOS_FRAMEWORK_SCHEME="AFNetworking iOS"
AF_TVOS_FRAMEWORK_SCHEME="AFNetworking tvOS"
AF_OSX_FRAMEWORK_SCHEME="AFNetworking OS X"

AF_IOS_EXAMPLE_SCHEME="iOS Example"
AF_TVOS_EXAMPLE_SCHEME="tvOS Example"
AF_OSX_EXAMPLE_SCHEME="OS X Example"
```

Note these are static values that will be used downstream in other .env files. These variables are _not_ used directly by the AFNetworking fastlanes.

### .env.default
The `.env.default` file is loaded second, and should define default values for your configuration. Note that if you do not use the `--env` flag when running the fastlane, these definitions should allow the tests to properly run.

```
AF_IOS_SDK=iphonesimulator9.1
AF_MAC_SDK=macosx10.11
AF_TVOS_SDK=appletvsimulator9.0

AF_CONFIGURATION=Release

#Specific Environment Variables needed for the Scan action
#Run `fastlane action scan` to see all available environment variables.
SCAN_WORKSPACE=$AF_WORKSPACE
SCAN_SCHEME=$AF_IOS_FRAMEWORK_SCHEME
SCAN_DESTINATION="OS=9.1,name=iPhone 6s"
SCAN_SDK=$AF_IOS_SDK
SCAN_OUTPUT_DIRECTORY=fastlane/test-output

#Specific Environment Variables needed for the Xcode build action. 
#Run `fastlane action xcodebuild` to see all available environment variables.
EXAMPLE_WORKSPACE=$AF_WORKSPACE
EXAMPLE_SCHEME=$AF_IOS_EXAMPLE_SCHEME
EXAMPLE_DESTINATION=$SCAN_DESTINATION
```

Note that when a new version of Xcode is released, the SDK's can simply be bumped here, and will automatically be picked up by fastlane.

### .env.{CUSTOM}
Finally, a custom environment can be defined to override any previous environment variables that have been set, or set additional environment variables needed for the specific job. AFNetworking creates a custom environment variable file for each platform/version combination tests should be run on. For example, here is the `.env.ios91` env file:

```
SCAN_DESTINATION="OS=9.1,name=iPhone 6s"
EXAMPLE_DESTINATION=$SCAN_DESTINATION
```

Note the only thing that is changed is the destination. To test multiple platform/OS versions, we simply do the following:

```
fastlane test_framework --env ios91
fastlane test_framework --env ios90
fastlane test_framework --env ios84
fastlane test_framework --env tvos90
etc....
```

## CI

Configuring your repository for continuous integration allows your project to constantly be under test, and gives you greater confidence in your release. Both [Travis CI](http://docs.travis-ci.com/user/getting-started/) and [Circle CI](https://circleci.com/docs/getting-started) offer free plans for open source projects. This guide will focus on Travis CI integration.

Here is an example `.travis.yml` file that tests the framework, and builds the example project, using the environment variable files defined in the previous section. Travis CI iterates over the Matrix, and creates unique jobs for each line listed, allowing multiple platform/OS version combinations to be tested.

```
language: objective-c
osx_image: xcode7.1
sudo: false
env:
  global:
  - LC_CTYPE=en_US.UTF-8
  - LANG=en_US.UTF-8
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
  - fastlane ci_commit configuration:Debug --env $FASTLANE_ENV
  - fastlane ci_commit configuration:Release --env $FASTLANE_ENV
```

# Next Steps
Once testing is setup, you are ready to setup [automated deployment](DeploymentGuide.md).