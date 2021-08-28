# Code coverage

## Slather
Is ideal for getting reports on the amount of code coverage in an xcode project.
* Requires that you have first run the unit tests as Slather only *parses* the data
* Does not work with Swift Packages at the time of writing this.

#### Installation
```
gem install slather
```
#### Example Usage:
```
boyceestes@ip-192-168-1-157 CoreDataDemo1 % slather coverage --scheme CoreDataDemo1 --show ./CoreDataDemo1.xcodeproj/
Slathering...
CoreDataDemo1/CoreData/Movie.swift: 25 of 38 lines (65.79%)
CoreDataDemo1/CoreData/StorageProvider.swift: 135 of 211 lines (63.98%)
CoreDataDemo1/Source/MovieProvider.swift: 23 of 35 lines (65.71%)
CoreDataDemo1/Source/Utilities/Constants.swift: 0 of 12 lines (0.00%)
CoreDataDemo1/Source/Views/CoreDataDemo1App.swift: 6 of 6 lines (100.00%)
CoreDataDemo1/Source/Views/MoviesView.swift: 12 of 24 lines (50.00%)
Tested 201/326 statements
Test Coverage: 61.66%
Slathered
```

#### Fastlane


## CodeCov