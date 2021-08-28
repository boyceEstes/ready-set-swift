# Using Swift packages

* [Add package to project](#add-package-to-project)
	* [Adding packages in development](#adding-packages-in-development)
* [Accessing your resources](#accessing-your-resources)


## Add package to project
Apparently there are a few different ways that you can add a Swift Package to your project.

Way numero uno:
* File > Swift Packages > Add Package Dependency...
* Specify the package URL and the minimum version number.
	* There are some fancy ways to specify branches/commits but I haven't explored it yet.

Alternatively:
* I saw that you can simply drag and drop your package's root directory into an Xcode project. I still need to try this out. But I believe that it will give the option to specify versions after you do this and just kind of do the magic like in the first method.


Thirdly,
* Navigate to `xcodeproj` file
* Select the 'Swift Packages' tab
* Select 'Plus' button below the list of packages


Make sure that your package's Library has been added to your project. One way to do this is by 
* Click on your project in the *Project navigator*, the `.xcodeproj` file
* Select your target.
* Navigate to the *General* tab.
* Make suer that you see your Library in the section that says 'Frameworks, Libraries, and Embedded Content'.

* You should also see your dependency if you navigate to your target's *Build Phases* tab. It should be included in the section titled 'Link Binary With Libraries'

To actually use the package, you need to make sure that you have imported it into the file that you want to use it in. `import CoreDataStorage`. 


### Adding packages in development
Whenever you are creating packages for your applications, you might not want to tag each of them in order to actually use them with some minimum version number.

Instead you can specify a branch that you wish for Xcode to look at for your package path. Whenever you call to update your package, Xcode will go to the specified branch in the git repository that you have specified the package is in. It will look for the latest commit and then it will upload it.

You can actually go even further than that. If you want to test changes before you even push a commit, you can specify to look at package versions based on a *commit* instead of a branch. 
* Full disclosure, I haven't tried the commit version, but it should be similar to the branch strategy. Xcode would look at a commit and try to get any changes to the one specified.


### Modifying package versioning
Sometimes you would want to change from looking at a specific branch for your Swift package updates, which is great for development, to an exact version number, which is better for releases.

* Navigate to `xcodeproj` file
* Select the 'Swift Packages' tab
* Select your new package 
<br />



## Accessing your resources
If you have resources in a package, you clearly need to be able to access them. 

When Xcode builds your Swift package, it treats each target as a module. If a module includes resources, it will create a resource bundle and an internal static extension on *Bundle* to access it for each module.

Use the extension to locate package resources:
```
let settingsURL = Bundle.module.url(forResource: "settings", with Extension: "plist")
```
* **Note**: Always use `Bundle.module` to access the resources as the path to the dependency could change.

If you want to make a package available to apps that depend on your Swift package, declare a public constant for it. (Like was done above (at least as long as it is public))
<br />
