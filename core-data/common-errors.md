# Common errors

* [Property cannot be marked @NSManaged because its type cannot be represented in Objective-C](#property-cannot-be-marked-nsmanaged-because-its-type-cannot-be-represented-in-objective-c)
* [An NSManagedObject of class 'NSManagedObject' must have a valid NSEntityDescription](#an-nsmanagedobject-of-class-nsmanagedobject-must-have-a-valid-nsentitydescription)

## Property cannot be marked @NSManaged because its type cannot be represented in Objective-C

This is mentioned in [Creating NSManaged properties](./subclassing-nsmanagedobject#creating-nsmanaged-properties) but the reason this is happening is because `NSManagedObject` utilizes runtime features of Objective-C. This means that all the attributes of the subclass must be representable in Objective-C.

If you want an attribute Optional, you may ned to change the type from something like `Double` to `Decimal` or `NSNumber`.

`Set<GenericType>` values are still okay to use optionally.


## An NSManagedObject of class 'NSManagedObject' must have a valid NSEntityDescription
### First occurence
**SOLVED**
#### On: 
```
try mainContext.execute(batchInsert) // Specifically 'execute'
```
#### Error:
```
"An NSManagedObject of class 'NSManagedObject' must have a valid NSEntityDescription."
```
#### Solution:
* Set the `Class Module`'s namespace in the Core Data Model editor to `Current Product Module`

### Second occurence
**SOLVED**

#### On:
* Import the project source module using `@testable import CoreDataDemo1`
* This happened specifically whenever trying to call the batch method. Other non-batch insert methods were made that performed fine without this error.
```
func test_parseAndSave_batchInsert_fastest() {

    let storageProvider = StorageProvider(storeType: .inMemory)

    self.measure {

        storageProvider.parseMovieDataFromJSON(with: .batchInsert)
    }
}
```

#### Error(s):
```
warning: Multiple NSEntityDescriptions claim the NSManagedObject subclass 'CoreDataDemo1.Movie' so +entity is unable to disambiguate.

CoreData: error: +[CoreDataDemo1.Movie entity] Failed to find a unique match for an NSEntityDescription to a managed object subclass

Failed to read dyld info for process 85766 (5)
<unknown>:0: error: -[CoreDataDemo1Tests.StorageProviderTests test_parseAndSave_batchInsert_fastest] : An NSManagedObject of class 'NSManagedObject' must have a valid NSEntityDescription. (NSInvalidArgumentException)

An NSManagedObject of class 'NSManagedObject' must have a valid NSEntityDescription. (NSInvalidArgumentException)
```

#### Solution:
The problem was that the the ManagedObjectModel was being loaded in multiple times. Once for the actual project StorageProvider, and then it was being loaded one more time for the Mock instance that we wanted to use.

Normally I'm not sure that this would be a problem since it ran fine on the other non-batch Core Data interactions, but the compiler must be stricter with these statements. 

The **solution** put in place to prevent this was to cache the Managed Object Model after it loaded the first time. We will use this in-memory instance whenever we want to initialize a new Persistent Container later (for unit testing in this case).
```
static var model: NSManagedObjectModel?


static func model(name: String) throws -> NSManagedObjectModel {

    if model == nil {
        model = try loadModel(name: name, bundle: Bundle.main)
    }
    return model!
}


// CoreDataError is a basic error enum with only these two cases right now.
static func loadModel(name: String, bundle: Bundle) throws -> NSManagedObjectModel {

    let name = StorageProvider.modelName
    guard let modelURL = bundle.url(forResource: name, withExtension: "momd") else {
        throw CoreDataError.modelURLNotFound(forResourceName: name)
    }

    guard let model = NSManagedObjectModel(contentsOf: modelURL) else {
        throw CoreDataError.modelLoadingFailed(forURL: modelURL)
    }

    return model
}
```
These new methods are called from:
```
// Load Core Data Persistent Container
let name = StorageProvider.modelName
persistentContainer = NSPersistentContainer(name: name, managedObjectModel: try! StorageProvider.model(name: name))
```
* Clearly this is not the safest way to do this, meant to show the concept. These snippets can be cleaned up for sure.