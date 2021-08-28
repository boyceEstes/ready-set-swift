# Batching

Batching is the best way to insert a bunch of raw data directly to the persistent store if you're not worried about any relationships. The only downside is that you will need to manually trigger merges to any contexts currently have the data.

* [Batch insert](#batch-insert)
* [Batch delete](#batch-delete)

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

* We can make it generic like the following:
```
    // Generic batch insert for whatever array of objects.
    public func batchInsertData<T>(_ insertData: [T], for entityName: EntityName) {

        guard !insertData.isEmpty else { return }

        // We could probably make this process of merging changes to context generic.
        var batchInsertRequest: NSBatchInsertRequest?

        switch entityName {
        case .movie:
            guard let movieData = insertData as? [String] else { return }
            batchInsertRequest = Movie.newBatchInsertRequest(with: movieData)

        default:
            fatalError("Unknown entity. Cannot perform batch insert request")
        }

        guard let batchInsertRequest = batchInsertRequest else { return }

        batchInsertRequest.resultType = .objectIDs
        do {
            let result = try mainContext.execute(batchInsertRequest) as? NSBatchInsertResult
            let insertedObjectIDs = result?.result as? [NSManagedObjectID]

            // Create dictionary to hold all managed objects that were inserted in persistent store
            let changes: [AnyHashable: Any] = [NSInsertedObjectIDsKey: insertedObjectIDs ?? []]

            // merge changes to all contexts that might have managed objects in memory
            NSManagedObjectContext.mergeChanges(fromRemoteContextSave: changes, into: [mainContext])
        } catch {
            assertionFailure("Failed to batch insert movie data with error: \(error)")
        }
    }
```
<br />


## Batch delete
* Best for large deletions with very large data sets.
* This runs faster than deleting Core Data entities yourself in code because batches operate directly in the persistent store at the SQL level. 
* No need to load the objects into memory then delete and then save back to disk.
* Since objects are not loaded into memory, anything deleted in a batch request is not reflected in the context automatically. Therefore you need to manually remove objects in memory that have been deleted from the persistent store after a batch Delete.
* **NOTE**: ALL validation rules that normally act on relationships during delete like cascading are not enforced during batch deletions. You will need to delete manually to handle relationsips.
<br />


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



#### Sources
* [Apple documentation](https://developer.apple.com/library/archive/featuredarticles/CoreData_Batch_Guide/BatchDeletes/BatchDeletes.html)
* [Avanderlee NSBatchDeleteRequest](https://www.avanderlee.com/swift/nsbatchdeleterequest-core-data/)
* [Cocoacasts delete Core Data entities](https://cocoacasts.com/how-to-delete-every-record-of-a-core-data-entity)