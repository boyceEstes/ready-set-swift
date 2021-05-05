# Basics

See CoreDataDemo1 project for more details

* [Common errors](#common-errors)
* [Batch insert](#batch-insert)
* [Classic delete](#classic-delete)
* [Batch delete](#batch-delete)
* [Measure performance](#measure-performance)

## Common errors

## Batch insert
* Have all of the information needed to make all records on hand and ready. Something like `[MovieData]`.

* Each BatchInsertRequest must be specific to the given entity you are inserting because we need to set their exact properties.

* Example:
```
private func newBatchInsertRequest(with movieData: [String]) -> NSBatchInsertRequest {
    
    var index = 0
    let total = movieData.count
    
    // This is a recursive method. Return false until you want it to end.
    let batchInsert = NSBatchInsertRequest(entity: Movie.entity(), managedObjectHandler: { managedObject -> Bool in
        
        // Only finish the loop whenever we are at or above our total data
        guard index < total else { return true }
        
        if let movie = managedObject as? Movie {
            movie.name = movieData[index]
        }
        
        index += 1
        return false
    })
    
    return batchInsert
}
```

* Execute the batch insert directly through the managed object context:
```
private func insertMovieData(_ movieData: [String]) {
    
    guard !movieData.isEmpty else { return }
    
    let batchInsert = newBatchInsertRequest(with: movieData)
    
    do {
        try mainContext.execute(batchInsert)
    } catch {
        assertionFailure("Failed to batch insert movie data with error: \(error)")
    }
}
```
* This is possible to make the above call generic... I think. Instead of taking in an array of strings we would take in an array of any object.
<br />


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


## Batch delete
* Best for large deletions with very large data sets.
* This runs faster than deleting Core Data entities yourself in code because batches operate directly in the persistent store at the SQL level. 
* No need to load the objects into memory then delete and then save back to disk.
* Since objects are not loaded into memory, anything deleted in a batch request is not reflected in the context automatically. Therefore you need to manually remove objects in memory that have been deleted from the persistent store after a batch Delete.
* **NOTE**: ALL validation rules that normally act on relationships during delete like cascading are not enforced during batch deletions. You will need to delete manually to handle relationsips.
<br />


#### Set up a batch delete with a fetch request. 
* Requires `NSFetchRequest<NSFetchRequestResult>` instead of the usual `NSFetchRequest<EntityName>` object type. 
* Batch fetch requests can still be sorted and use predicates like any other fetch request.
```
let fetchRequest = NSFetchRequest<NSFetchRequestResult>(entityName: "Content")
let batchDeleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
```

#### Executing batch request
* Use managed object context to execute.
* `execute(_:)` is a throwing function so you must enclose it in do-try-catch block
* executing a request will return return a response when successful
    * If `batchDeleteRequest.resultType` is default value (`.resultTypeStatusOnly`), then it returns nothing.
    * If `batchDeleteRequest.resultType` is `.resultTypeObjectIDs`, it will return a list of the `NSManagedObjectID`s for the instances that were deleted.
```
let batchDeleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
batchDeleteRequest.resultType = .resultTypeObjectIDs

do {
    // result will be list of NSManagedObjectIDs of deleted instances
    let result = try moc.execute(batchDeleteRequest)
} catch {
    fatalError("Failed to execute delete request with error: \(error)")
}
```

#### Updating application after execution
* Only do this is your application has objects loaded into memory that were deleted. Otherwise nothing will happen.
* Make sure to set resultType so we can return the IDs of the deleted objects as shown above.
* Call `mergeChangesFromRemoteContextSave` so that all `NSManagedObjectContext` instances that are referenced will be notified that the list of entities with the given `NSManagedObjectID` have been deleted.
```
do {
    // Execute the batch delete request
    let result = try mainContext.execute(batchDeleteRequest) as? NSBatchDeleteResult
    let deletedObjectIDs = result?.result as? [NSManagedObjectID]
    
    // Create dictionary to hold all managed objects that were deleted in persistent store
    let changes: [AnyHashable: Any] = [NSDeletedObjectIDsKey: deletedObjectIDs ?? []]
    
    // merge changes to all contexts that might have managed objects in memory
    NSManagedObjectContext.mergeChanges(fromRemoteContextSave: changes, into: [mainContext])
    
} catch {
    assertionFailure("CoreData failed to batch delete with error: \(error)")
}
```

#### TL;DR
* "Proper" batch delete example
* Make sure that you are merging changes with all context's that might have stale data after you delete
* Make sure that what you are deleting does not have any relationships
```
    public func batchDelete() {
        
        // Create fetch request
        let fetchRequest = NSFetchRequest<NSFetchRequestResult>(entityName: Movie.entityName)
        
        // Create Batch Delete Request
        let batchDeleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
        batchDeleteRequest.resultType = .resultTypeObjectIDs
        
        do {
            // Execute the batch delete request
            let result = try mainContext.execute(batchDeleteRequest) as? NSBatchDeleteResult
            let deletedObjectIDs = result?.result as? [NSManagedObjectID]
            
            // Create dictionary to hold all managed objects that were deleted in persistent store
            let changes: [AnyHashable: Any] = [NSDeletedObjectIDsKey: deletedObjectIDs ?? []]
            
            // merge changes to all contexts that might have managed objects in memory
            NSManagedObjectContext.mergeChanges(fromRemoteContextSave: changes, into: [mainContext])
            
        } catch {
            assertionFailure("CoreData failed to batch delete with error: \(error)")
        }
    }

```

#### Sources
* [Apple documentation](https://developer.apple.com/library/archive/featuredarticles/CoreData_Batch_Guide/BatchDeletes/BatchDeletes.html)
* [Avanderlee NSBatchDeleteRequest](https://www.avanderlee.com/swift/nsbatchdeleterequest-core-data/)
* [Cocoacasts delete Core Data entities](https://cocoacasts.com/how-to-delete-every-record-of-a-core-data-entity)