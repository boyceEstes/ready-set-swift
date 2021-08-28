# Basics

* [Add Pod](#add-pod)
* [Remove Pod](#remove-pod)
* [Uninstall CocoaPods from Xcode project](#uninstall-cocoapods-from-xcode-project)
* [General](#general)
* [Commands](#commands)
	* [Pod install](#pod-install)
	* [Pod outdated](#pod-outdated)
	* [Pod update](#pod-update)
* [Behind the scenes](#behind-the-scenes)
* [Sources](#sources)



## Add Pod
* Add the new pod to your `Podfile` file
	* You specify the target that you want the dependency for - If you wanted to use the pod in a Test target, for example, you would need to add this pod to that target as well: `target 'MyAppTests'`
Ex:
```
target 'MyApp' do
  pod 'AFNetworking', '~> 3.0'
  pod 'FBSDKCoreKit', '~> 4.9'
end
```
* Use each time you to retrieve pods for the project that you have specified in the Podfile.

Use command:
`pod install`
<br />



## Remove Pod
* Remove the pod from your Podfile

Use command:
`pod install`
<br />



## Uninstall CocoaPods from Xcode project

[Stackoverflow](https://stackoverflow.com/questions/16427421/how-to-remove-cocoapods-from-a-project)
<br />



## General
* The first line of a `Podfile` should specify the platform and version supported: 
```
platform :ios, '9.0'
```
* Open `MyApp.xcworkspace` to access your project with the pod dependencies.
* Always commit your `Podfile` to your shared repository.
* Always commit your `Podfile.lock` to your shared repository.
	* This is necessary because there are situations where even running `pod install` could give the members of the team different versions of the same dependency.
	* Ex:
	* User1 works alone installs Pod A with version 1.0 with `pod install` and then doesn't push the `Podfile.lock` to their repository. 
	* Months go by and now another developer joins the team, User2.
	* User2 clones the repository and uses `pod install` to install the necessary dependencies. However, since User1 installed Pod A developers released a new version, 1.1, so this is what User2 installs.
	* Now User1 and User2 look like darn fools because they have different versions of the same dependency. 
	* This could have been easily avoided by simply commiting the `Podfile.lock`. A real shame to see.
<br />



## Commands

#### Pod install
`pod install`
* Use anytime you want to add or remove new pods to your project.
* Downloads and installs new pods that are specified in the Podfile.
* This will write the version of each pod and *lock* this version in the `Podfile.lock` file.
* This command will only resolve dependencies for pods not already listed in `Podfile.lock` so that no work is repeated.
	* For pods listed in `Podfile.lock`, it downloads the explicit version listed in the lock file without trying to check if a newer version is available
* The first time this command is used, it will create the `.xcworkspace`


#### Pod outdated
`pod outdated`
* Lists all pods which have newer versions than the ones listed in `Podfile.lock`.


#### Pod update
`pod update`
`pod update PODNAME`

* The dependency specified by `PODNAME` will update to the latest version possible while still matching the restrictions for it in `Podfile`
* This will not take into account what is already in `Podfile.lock` - meaning it will still run even if the version is already the latest.
* If no `PODNAME` is specified, all pods will be updated.
<br />


## Behind the scenes
1. Creates or updates a workspace
2. Adds your project to the workspace if needed
3. Adds the CocoaPods static library project to the workspace if needed
4. Adds libPods.a to: targets => build phases => link with libraries.
5. Adds the CocoaPods Xcode configuration file to your app's project.
6. Changes your app's target configurations to be based on CocoaPods's.
7. Adds a build phase to copy resources from any pods you installed to your app bundle. i.e. a 'Script build phase' after all other build phases with the following:
	* Shell: `/bin/sh`
	* Script: `${SRCROOT}/Pods/PodsResources.sh`

Steps 3 onwards are skipped if the CocoaPods static library is already in your project.
<br />

## Sources
* [Using CocoaPods documentation](https://guides.cocoapods.org/using/using-cocoapods.html)
* [Install/Update documentation](https://guides.cocoapods.org/using/pod-install-vs-update.html)
