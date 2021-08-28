# Basic Xcode terminal commands

## Running tests
Navigate to the root directory of your xcode project in terminal. 

Type the below command, replacing 'CoreDataDemo1' for your own project/scheme name.
```
xcodebuild -project CoreDataDemo1.xcodeproj \
-scheme CoreDataDemo1 \
-destination platform=iOS\ Simulator,OS=14.5,name=iPhone\ 12 \
clean test | xcpretty
```
* **NOTE**: You must make sure that your simulator is available. You can check the list of simulators on your machine with this command: `xcrun simctl list`.
	* Also make sure that there are not any spaces between your commas in the command. It doesn't like that.

### CocoaPods
If you are using CocoaPods, the command will need to be adjusted to work with the xcworkspace instead of the xcodeproj.
```
xcodebuild -workspace CoreDataDemo1.xcworkspace \
-scheme CoreDataDemo1 \
-destination platform=iOS\ Simulator,OS=14.5,name=iPhone\ 12 \
clean test | xcpretty
```