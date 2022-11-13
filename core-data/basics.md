# Basics

See CoreDataDemo1 project for more details

* [Setting up Core Data stack](#setting-up-core-data-stack)
* [Create fetch requests](#create-fetch-requests)
* [Classic delete](#classic-delete)
* [Measure performance](#measure-performance)

# Persistence Container

# Create fetch requests

This is necessary for quering any Core Data information. You can attach `NSPredicate` or `NSSortDescriptor` to get very specific queries. But the foundational knowledge you need is to simply create a request for your entity. 

## TLDR

Create helper computed variable in your Core Data file like this
```
var fetchRequest: NSFetchRequest<Movie> {
    NSFetchRequest<Movie>(entityName: "Movie")
}
```

Call site:

```
let fetchRequest = Movie.fetchRequest
```

## Standard
This is the basic way to do it.
```
let fetchRequest = NSFetchRequest<Movie>(entityName: "Movie")
```

## Auto-generated
Whenever you generate an NSManagedObject subclass automatically, they will provide you with a method (I'm doing this from memory so it might not be 100%)

```
func fetchRequest() -> NSFetchedRequest<Movie>() {
    return NSFetchRequest<Movie>(entityName: "Movie")
}
```
It seems nice in theory, but last I checked it has a naming conflict because `fetchRequest()` is already a method name

So you would need to do something like this when using the method

```
let movie: NSFetchRequest<Movie> = Movie.fetchRequest()
```

Which is not really even a convenience method anymore. Instead, try renaming the method, or better yet, using the computed variable style that was shown above.

```
var fetchRequest: NSFetchedRequest<Movie> {
    return NSFetchRequest<Movie>(entityName: "Movie")
}
```

# Classic delete
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




# Sources
- [Cocoacasts delete Core Data entities](https://cocoacasts.com/how-to-delete-every-record-of-a-core-data-entity)
