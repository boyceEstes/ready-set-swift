# Github Actions
<!-- TOC -->

- [Github Actions](#github-actions)
	- [Glossary](#glossary)
	- [Notes](#notes)
	- [Get Started](#get-started)
	- [Specific for testing a Swift package](#specific-for-testing-a-swift-package)
	- [Workflow status badge](#workflow-status-badge)
		- [Exmaples](#exmaples)
  - [Sources](#sources)

<!-- /TOC -->

# Glossary
* **Workflow**: configurable automated process made up of one or more jobs. Defined by YAML file.
* **Job**: Set of steps that perform individual tasks and steps that can run commands or use an action.

# Notes
* Workflows can be triggered from a pull request or push to a specific/any branch. It can also be triggered with tags, scheduled, and some other ways.

# Get Started
1. Navigate to the repository's `Action` tab in Github.
2. Setup workflow
	* You should be editing a YAML file in the `/.github/workflows` directory.
3. After you submit some `actions.yml` file in `workflows` you can try to add it to the project.
	1. Start Commit
	2. Insert title
	3. Select "Create a new branch for this commit and start a pull request"
	* Committing the workflow file to a branch in your repository triggers the `push` event in the YAML file and runs your workflow.
	4. Finally select `Propose new file`
	5. Create Pull Request

* After the pull request has been made it will immediately start the validation process and you can see it in the PR. You can go to details to see the specific processes that are running, which are based on the commands specified in the YAML file.
	* In this case we just spit out information related to our project and the build machine with `echo` statements.
* You can view the Actions that were run (similar to how you would view validations in TeamCity) by navigating to the `Actions` tab.
	* You can select each run that occurs by selecting the item in `All workflows`.
	* After selecting a workflow, you can look at specific parts of the job including duration, billable time, the trigger, etc.
	* When you click the job in `Jobs` on the left pane, you can see each detailed step in the process.


## Specific for testing a Swift package
```
name: Swift

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: macos-latest

    steps:
    - uses: actions/checkout@v2

    - name: Build
      run: swift build -v

    - name: Run tests
      run: swift test -v

```

1. **Name**: Each `- name:` identifies a new command that you want the GitHub server to run. In this case we want to run two commands.
2. **Run**: The command that is performed by the step. This is analagous to the command that you can use in terminal

The same rules apply for the second command "Run Tests". Its name is identified as "Run tests" and the command that we want the build machine to use is `swift test -v`.

**NOTE**: More experimentation will need to be done, but an easy way to create more fine-grained testing scenarios is to separate your tests into multiple test targets in your `Package.swift` file (assuming youre using GitHub actions on a swift package, of course). Then you can specify the test target you would like to run tests for, if you only wanted certain tests performed at a certain time.

Example of a workflow action that would build some swift package for with the macOS, and then build and test for specific iOS device schemes

```
jobs:
  unit_tests:
    runs-on: macos-latest
    steps:
    - name: Repository checkout
      uses: actions/checkout@v2

    - name: Lint
      run: swiftlint

    - name: Build for macOS
      run: swift build -v

    - name: Run macOS tests
      run: swift test -v

    - name: Build for iOS
      run: set -o pipefail && env NSUnbufferedIO=YES xcodebuild build-for-testing -scheme IndexedDataStore -destination "platform=iOS Simulator,OS=latest,name=iPhone 12" | xcpretty

    - name: Run iOS tests
      run: set -o pipefail && env NSUnbufferedIO=YES xcodebuild test-without-building -scheme IndexedDataStore -destination "platform=iOS Simulator,OS=latest,name=iPhone 12" | xcpretty
```
This allows us to have more control whenever we are creating a CI pipeline because we can have certain schemes test certain test targets. 

For instance, we might not always want to run tests on the End-to-End API tests or fully integrated Cache tests since they are more time intensive. 

But we might want to run it before we merge to the main branch

# Workflow status badge
Whenever you have testing implemented with some CI, you can usually attach a pretty badge to show off your brilliant passing project.

A status badge shows whether the workflow is currently failing or passing and usually goes on the project's `README.md` file.

By default, the badge displays the status of your default branch, however this can be customized to show whatever branch or event you want by using parameters.

General template:
```
![example workflow](https://github.com/<OWNER>/<REPOSITORY>/actions/workflows/<WORKFLOW_FILE>/badge.svg)
```
## Exmaples
Default:
```
![example workflow](https://github.com/github/docs/actions/workflows/main.yml/badge.svg)
```

Branch parameter:
```
![example branch parameter](https://github.com/github/docs/actions/workflows/main.yml/badge.svg?branch=feature-1)

```

Event parameter:
```
![example event parameter](https://github.com/github/docs/actions/workflows/main.yml/badge.svg?event=pull_request)
```

# Sources
- [Running tests in Swift package with GitHub actions – Augmented Code](https://augmentedcode.io/2021/04/26/running-tests-in-swift-package-with-github-actions/)