# Basics

See CoreDataDemo1 project for more details

* [Passing entities through parameters](#passing-entities-through-parameters)
* [Classic delete](#classic-delete)
* [Setting up Core Data stack](#setting-up-core-data-stack)
* [Create fetch requests](#create-fetch-requests)
* [Measure performance](#measure-performance)

## Passing entities through parameters
Sometimes we want to pass an ambiguous managed object to a function that is abstracted to work on *any* managed object. The easy way to do this is by specifying `NSEntityDescriptor` in the parameter and passing the managed objects entity description in the argument like `Movie.entity()`.

An example I found myself using this for is to batch insert something.
```
    // Generic batch insert for whatever array of objects on some Entity
    public func batchInsertData<T>(_ insertData: [T], for entityDescription: NSEntityDescription) {

        // Get the name of the entity
        guard !insertData.isEmpty,
              let entityName = entityDescription.name else { return }

        var batchInsertRequest: NSBatchInsertRequest?

        // Create batch insert based on the entity's static property "entityName"
        switch entityName {
        case Movie.entityName:
            guard let movieData = insertData as? [String] else { return }
            batchInsertRequest = Movie.newBatchInsertRequest(with: movieData)

        default:
            return
        }

        guard let batchInsertRequest = batchInsertRequest else { return }

```

* An alternative worth considering is to create a string enum for all of the names like ManagedObjectNames and pass that in.
    * Pros:
        * If we make it a String enum we would directly have the name from what we pass and its more type-safe than a string.
        * This would allow us to not need the default case in the switch state because our list would cover all enums (entities) possible.
    * Cons:
        * Passing NSEntityDescription seems like THE accepted way to get "some" managed object type.
        * We would need to have need to save all of the names of the managed objects somewhere other than a static variable on their class. This isn't terrible in itself, but it is nice to have that easy way to view its name upon looking at its class.
        * `Movie.entityName` is less typing than `EntityName.movie.rawValue`


## Classic delete
* Best for dealing with relatively small, more intricately relating data sets.
* Load objects into memory -> delete objects from context -> save to persistent coordinator
* This takes much longer since it will need to read and write to disk, however, it has the benefit of being able to automatically update the changes AND it will delete relationship attributes based on the validation rules set in the Core Data model.
* To reduce memory footprint, you can only choose to fetch the `NSManagedObjectID` instead of the full managed object.
    * You can set `request.resultType` to `.managedObjectIDResultType` (Donny Wals does it this way)
    * You can set `request.includesPropertyValues` to `false` - seems to be the more tried and true way I've seen around.
    * Both seem to accomplish the same thing.
```
    public func deleteObjectsById() {

        // Do not include property values - full managed object
        // Alternative could be to declare as NSFetchRequest<NSManagedObjectID> and then set request.resultType = .managedObjectIDResultType

        let fetchRequest = Movie.createFetchRequest()
        fetchRequest.includesPropertyValues = false

        do {
            let movies = try mainContext.fetch(fetchRequest)

            for movie in movies {
                mainContext.delete(movie)
            }

            save(mainContext)
        } catch {
            assertionFailure("CoreData failed to delete with error: \(error)")
        }
    }
```

#### Sources
[Cocoacasts delete Core Data entities](https://cocoacasts.com/how-to-delete-every-record-of-a-core-data-entity)
<br />


## Setting up Core Data stack



## Create fetch requests
It would be nice to have a shortcut to create fetch requests for each of the managed object entities.

* [Standard](#standard)
* [Auto-generated](#auto-generated)
* [Custom](#custom)
* [Managed object type](#managed-object-type)

### Standard
This is the default way to do it.
```
let fetchRequest = NSFetchRequest<Movie>(entityName: "Movie")
```

### Auto-generated
Whenever you generate an NSManagedObject subclass automatically, they will provide you with a method (I'm doing this from memory so it might not be 100%)
```
func fetchRequest() -> NSFetchedRequest<Movie>() {
    return NSFetchRequest<Movie>(entityName: "Movie")
}
```

### Custom
It is easy to shorten this by creating a static method on each managed object subclass.
```
class Movie: NSManagedObject {

    // ...

    public static func createFetchRequest() -> NSFetchRequest<Movie> {
        return NSFetchRequest<Movie>(entityName: "Movie")
    }
}
```

### Managed Object Type
There is a protocol that I found in the Halo project that kinda abstracts key parts of managed objects.
```
protocol ManagedObjectType: AnyObject {

    static var entityName: String { get }
}


extension ManagedObjectType {

    static public func createFetchRequest<T: NSManagedObject>() -> NSFetchRequest<T> {

        return NSFetchRequest<T>(entityName: entityName)
    }


    // So far I've only noticed that this is useful for Batching
    static public func createFetchRequestResult() -> NSFetchRequest<NSFetchRequestResult> {

        return NSFetchRequest<NSFetchRequestResult>(entityName: entityName)
    }
}

```

The key part of this is to notice the extension's createFetchRequest method. Ideally this would mean that any object that conforms to this protocol would have an easy static method to create a fetch request.

Unfortunately it is a little verbose:
```
let movieFetchRequest: NSFetchRequest<Movie> = Movie.createFetchRequest()
```

Andddd what's the point of this? It saves no typing from the [standard](#standard) way to call this.

Therfore. I would remove this method from the managed object type protocol:

```
protocol ManagedObjectType: AnyObject {

    static var entityName: String { get }
}


extension ManagedObjectType {

    // So far I've only noticed that this is useful for Batching
    static public func createFetchRequestResult() -> NSFetchRequest<NSFetchRequestResult> {

        return NSFetchRequest<NSFetchRequestResult>(entityName: entityName)
    }
}
```
