# Testing Swift Package
Testing your swift package should be the funnest thing in the world. Easy. Simple. Beautiful.

We have the same ability as with normal Xcode projects to run unit tests on our package. Before doing that, the `Package.swift` file must be done and specify that we have a test target. (I believe. Full disclosure, I've never tried it without it.)

## Set up tests in your package
This section can be *completely* skipped. If you ever need to know some details on how to set up targets, this is your place, but Xcode makes it easy for us by doing all of this automatically so you don'tneed to worry about it at all.

### Add test target to package manifest

Example of a package manifest file that has been set up to have a testTarget.

```
let package = Package(
    name: "NetworkMe",
    products: [
        // Products define the executables and libraries a package produces, and make them visible to other packages.
        .library(
            name: "NetworkMe",
            targets: ["NetworkMe"]),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "NetworkMe",
            dependencies: []),
        .testTarget(
            name: "NetworkMeTests",
            dependencies: ["NetworkMe"]),
    ]
)

```

Here we have created a production target "NetworkMe" along with a single test target that is called "NetworkMeTests". It will need to use the production target as a dependency, but that is all that it takes!

### Add a test
You will need to add a directory to the `Tests/` directory in the package that matches the name that was specified in the package manifest's `testTarget`. In this case, there needs to be a structure like `Tests/NetworkMeTests/`. In this directory, you place any of the tests that you need to create.

## Run tests
There are various ways to run your unit tests after you have created your test target and implemented some tests.

* Any way that Xcode will normally let you run these.
  - `cmd+u`
  - Hold play button at the top and navigate to `Test`
  - Navigate to the Test Navigator in the left panel and select the play button that appears after each of the tests/targets.
* Programmatically starting via command line - This is done differently than in an Xcode project. Simpler.
  1. Start by navigating to the directory in terminal.
  2. Build the package: `swift build` (Use `-v` for more details)
  3. Test the package: `swift test` (Use `-v` for more details)


## Multilple test targets
There are sometimes that you could want multiple test targets in your package. For example, because end-to-end testing will take much longer than your unit tests, you could separate them into two different test targets.

After they are separated, it is possible to easily run one target and not the other. This allows more options while setting up your Continuous Integration.

To do this, you can modify the package manifest file by adding another testTarget, like so:
```
.target(
    name: "NetworkMe",
    dependencies: []),
.testTarget(
    name: "NetworkMeTests",
    dependencies: ["NetworkMe"]),
.testTarget(
    name: "NetworkMeAPIEndToEndTests",
    dependencies: ["NetworkMe"]),
```

Now we will need to set the structure in the `Tests` directory.

Next, enter whatever unit tests you want to use.

### Run only one test target
Whenever you use the command `swift test`, all test targets will be tested. But in this case, we would only want one of them to run.

We can filter the command so that it only applies to one test target by using the following command:
```
$ swift build -v
$ swift test --filter NetworkMeAPIEndToEndTests
```

And that's all there is to it!
