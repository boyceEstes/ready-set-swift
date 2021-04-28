# Basics

See CoreDataDemo1 project for more details

* [Common errors](#common-errors)
* [Batch insert](#batch-insert)
* [Measure performance](#measure-performance)

## Common errors

## BatchInsert
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