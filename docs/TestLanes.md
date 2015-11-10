# Test Lanes

The test lanes are meant to aid in testing frameworks in continuous integration environments, including running all tests and confirming the example project builds and runs

## ci_commit
Runs tests and builds example for the given environment

The lane to run by ci on every commit. This lanes calls the lanes `test_framework` and `build_example`.

###Example:

```
fastlane ci_commit configuration:Debug --env ios91
```

###Options

 * **`configuration`**: The build configuration to use. (`AF_CONFIGURATION`)


## test_framework
Runs all tests for the given environment

Set `scan` action environment variables to control test configuration

###Example:

```
fastlane test_framework configuration:Debug --env ios91
```

###Options

 * **`configuration`**: The build configuration to use.


## build_example
Builds the example file

Set [`xcodebuild`](https://github.com/fastlane/fastlane/blob/master/docs/Actions.md#xcodebuild) action environment variables to control build configuration

###Example:

```
fastlane build_example configuration:Debug --env ios91
```

###Options

 * **`configuration`**: The build configuration to use.