# Generic methods

Sometimes we want to pass an ambiguous managed object to a function that is abstracted to work on *any* managed object. 

The easy way to do this is by specifying `NSEntityDescriptor` in the parameter and passing the managed objects entity description in the argument like `Movie.entity()`.

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