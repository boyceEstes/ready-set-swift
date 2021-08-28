# Creating Swift Package

* [Get started](#get-started)
* [Initial Package contents](#initial-package-contents)
* [Configure the Package](#configure-the-package)
* [Add code](#add-code)
	* [Modifying access level](#odifying-access-level)
* [Add a dependency on another Swift Package](#add-a-dependency-on-another-swift-package)
* [Package resources](#accessing-your-resources)
* [Versioning](#versioning)
* [Creating a tag](#creating-a-tag)
* [Sources](#sources)

## Get started

Create a Swift Package
1. Open Xcode
2. Select File > New > Swift Package
3. Choose a name and select file location.
4. Done!

* Note: To add version control you can check "Create Git repository on my Mac" or manually navigate to the directory and use the classic `git init`.
<br />



## Initial Package contents
* **README.md**: What your package does
* **Package.swift** (*Package manifest*): Describes the configuration for the swift package. Uses Swift and PackageDescription framework to define:
	* Package name,
	* Products,
	* Targets,
	* Dependencies
	* More.
* **Soources**: Source files for the package. They are scoped per *Target*. A package can have several targets, and, as a convention, each target's code resides in its own subfolder.
* **Tests**: Unit test target subdirectory. Follows same convention as standard targets, each test target's code resides in its own subfolder in this directory.
<br />



## Configure the Package
Since there is no `.xcproject` or `.xcworkspace`, packages must rely on their folder structure and use the *package manifest* for addition configurations.

Most of the important bits are set up by default to allow the Package to run. Useful things to know regarding the package manifest:
* Describe if your package is a *product* here.
* Describe any dependencies here - packages that this package depends on.
* Targets would include all of the targets available in this package. If you target depends on another target, you would need to list that dependency here (as well as in the separate dependencies area) 
	* These dependencies would include resources that are necessary for the target.

"Simple example" by Apple:
```
// swift-tools-version:5.3
import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [
        .macOS(.v10_14), .iOS(.v13), .tvOS(.v13)
    ],
    products: [
        // Products define the executables and libraries a package produces, and make them visible to other packages.
        .library(
            name: "MyLibrary",
            targets: ["MyLibrary", "SomeRemoteBinaryPackage", "SomeLocalBinaryPackage"])
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "MyLibrary",
            exclude: ["instructions.md"],
            resources: [
                .process("text.txt"),
                .process("example.png"),
                .copy("settings.plist")
            ]
        ),
        .binaryTarget(
            name: "SomeRemoteBinaryPackage",
            url: "https://url/to/some/remote/binary/package.zip",
            checksum: "The checksum of the XCFramework inside the ZIP archive."
        ),
        .binaryTarget(
            name: "SomeLocalBinaryPackage",
            path: "path/to/some.xcframework"
        )
        .testTarget(
            name: "MyLibraryTests",
            dependencies: ["MyLibrary"]),
    ]
)
```

### Specify platform and version
In the example, we can see that they have a `platforms` section written out. This is not *necessary* for the package to become operation.

If you do not include package, the default is the oldest supported version for all of the platforms (iOS, macOS, etc.)

So if you need to use some API that requires a minimum deployment target version, you need to specify this.

```
    platforms: [
        .macOS(.v10_15), .iOS(.v13)
    ],
```

macOS(.v10_15) - Catalina
macOS(.v11) - Big Sur (not yet supported in some places - looking at you Github Actions)
<br />



## Add code
You would add source code the same way as any normal project. If you add a new file to the target subdirectory that you want, for example Add a Swift file to `Sources/CoreDataStorage`, it will be automatically included in your target.

### Modifying access level
Whenever you separate your code into separate modules, you will need to take some extra measures to make sure that it can communicate with each individual module. 

By default, Swift symbols are creating with an `internal` access level. This means that only other entities that are *internal* to the project can access the symbol. This goes for classes, structs, functions, variables, etc.

For example:
```
class BigTimeRushWasLowKeyFire() {

	init() {}
}
```
Needs to instead be:
```
public class BigTimeRushWasLowKeyFire() {

	public init() {}
}
```

* **Note**: For structs you will need to explicitly write the initializer so that it can be public scoped. Gross.

<br />



## Add a dependency on another Swift Package
Define this in the *package manifest*.

There are two main ways to retrieve dependency packages:
* Contain the path to resource locally in your package.
* Fetch the package via its URL.

```
dependencies: [    
    // Dependencies declare other packages that this package depends on.
    .package(url: "https://url/of/another/package.git", from: "1.0.0"),
    .package(path: "path/to/a/local/package/", "1.0.0"..<"2.0.0")],
```

* `Package.resolved` file will record the results of Xcodes *dependency resolution*, which figures out the exact version your package dependencies should/can be.
	* If you add a Swift package as a package dependency to an regular degular Apple application, you can find the `Package.resolved` file inside your `.xcodeproj` or `xcworkspace`.
<br />



## Distribute binaries as Swift Package
Swift Packages do not have to provide the source code files. Instead, you can choose to distribute binaries. If you want to sell a closed-source library, for example, you could make it available as a binary.
<br />



## Package resources
To add asset files as package resources to your Swift project, **declare a Swift tools version of 5.3 or later in your manifest file**.

For example, Swift packages can contain user interface comonents that use asset catalogs, storyboards, `.string` files, etc.

[More info](https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package)

### Including
You must explicitly declare or exclude some resources. When Xcode can't automatically handle resources, declare it as a resource in the *package manifest*. 

Here is an example that assumes `text.txt` resides in your `Sources/CoreDataStorage` directory and you want to include it as a package resource. In `Package.swift`:
```
targets: [
    .target(
        name: "MyLibrary",
        resources: [
            .process("text.txt")]
    ),
]
```

Two rules to determine how Xcode treats resources:
* Process rule
	* Use `process(_:localization:)` to apply this rule.
	* Xcode processes the resource according to the platform you're building the package for.
		* For example, Xcode might optimize image files for a platform that supports those optimizations.
		* If this is applied to a directory path (like `/tons-of-animals-directory/`, Xcode will recursively apply the rule to everything inside the directory.
* Copy rule
	* Resource file should remain untouched or have a certain directory structure for resources.
	* Use `copy(_:)` to apply this rule.

### Excluding
If you do not want a resource to travel along with the package resources, you can make sure it doesn't by excluding it from a target.
```
targets: [
    .target(
        name: "MyLibrary",
        exclude:["instructions.md"]
    ),
]
```

* Rule of thumb: Don't place files that aren't resources in a target's source folder. 
	* If that's not possible then group all of your files you *must* include together in a directory and exclude that whole directory instead each individual file.




## Versioning
When you are adding a Swift package to a project, you might notice the version specifier that you have to fill out. Well, where do you actually SET that when you're making the package?

Great question, happy I asked. 

You do it through Github(or whatever git provider) tags.
To version your package:
* When you commit, tag the last commit with a package version. Some three period-separated integer. `1.0.0`.

Whenever your product is in alpha it might be a good idea to just keep this before version `1.0.0`. I'd say to start small with `0.0.1` and work you way up with each commit (Probably a good thing to have done automatically).


### Creating a tag
This is pretty intuitive so don't freak out if you forget. 
* Just navigate to the repository on [github](https://github.com)
* On the right panel, *Releases*
* Choose 'Create a new release'
* Choose your branch
* Click the button to create the tag and it will create a tag for the last commit on that branch.

<br />


## Sources
* [Apple documentation - Swift Package Manager](https://docs.swift.org/package-manager/PackageDescription/PackageDescription.html)
* [Apple documentation - creating a standalone swift package](https://developer.apple.com/documentation/xcode/creating_a_standalone_swift_package_with_xcode)
