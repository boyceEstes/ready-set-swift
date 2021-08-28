# Testing Swift Package
Testing your swift package should be the funnest thing in the world. Easy. Simple. Beautiful.

We have the same ability as with normal Xcode projects to run unit tests on our package. Before doing that, the `Package.swift` file must be done and specify that we have a test target. (I believe. Full disclosure, I've never tried it without it.)

## TestTarget in Package Manifest

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
        .testTarget(
            name: "NetworkMeAPIEndToEndTests",
            dependencies: ["NetworkMe"]),
    ]
)

```
