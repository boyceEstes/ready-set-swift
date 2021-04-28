# Practical Core Data
(Danny Wols)

* [Must haves](#must-haves)
* [Creating PersistentContainer](#creating-persistentcontainer)
* [Define Core Data model](#define-core-data-model)
* [Managed Object](#managed-object)
* [Managed Object Context](#managed-object-context)
* [Adding new records](#adding-new-records)
* [Retrieving records](#retrieving-records)
* [Deleting records](#deleting-records)
* [Modifying records](#modifying-records)
* [Multithreading](#multithreading)
	* [Child contexts](#child-contexts)
	* [Responding to changes in another managed object context](#responding-to-changes-in-another-managed-object-context)
	* [Query generations](#query-generations)
* [Unit testing](#unit-testing)
	* [Setting up an in-memory SQLite store](#setting-up-an-in-memory-sqlite-store)

Search TODO for the sparkly bits that I want to test performance on.

### Must haves

In every project that utilizes Core Data here are a few "must haves" that will come in handy in one way or another.

* Background moc to save to the persistent store coordinator that has a child moc on the main thread to handle all things UI related.
* Save method for contexts that will properly propagate changes.
* Rollback saves with errors so that you know that you have removed whatever has caused the problem (althrough ideally you wouldn't have error - if this is happening frequently there is a problem to address)
* Create a StorageProvider (or whatever you want to call it) class that encapsulates the core data logic for the project.
	* Make sure that this provider is able to be mocked easily. This means that we can easily initialize a StorageProvider that is created with an "in-memory" Persistent Store type or file path.
* Turn on concurrency debugging for Core Data.
<br />


### Creating PersistentContainer

* requires model filename as an argument in initializer but that's it!

```
import CoreData

class StorageProvider {

	let persistentContainer: NSPersistentContainer

	init() {
		persistentContainer = NSPersistentContainer(name: "HelloCoreData")

		// Load persistent store of choice
		persistentContainer.loadPersistentStores(completionHandler: { description, error in
			guard error == nil else { 
				fatalError("Core Data failed to load: \(error)")
			}
		})
	}
}
```

* Failing to load the persistent store is a fatal error for a simple app. For a production app this could be caused by a few different things and should be handled differently. 
	* Fail could be caused by `NSFileReadNoPermissionError` or `NSFileWriteNoPermissionError`.
		* This could mean that your store is still encrypted (since your app is supposed to have permission to read/write files in its container).
		* If this happens try listenting to `.protectedDataDidBecomeAvailableNotification` and make a new attempt to load the persistent store when this is fired.

**Common Error**: `Failed to load model named YourModelName.`
* This will occur if there is a mismatch in the name provdied to the NSPersistentContainer initializer and your model resource. 
* Note: The compiler won't catch this error, it'll crash during runtime when you try to instantiate the container.

* Pass around this Storage Provider after is initialized somehow. Some popular choices are:
	* Sharables - `static var shared = StorageProvider()`
	* Dependency injection - pass in storage provider class to the view or controller from whatever calls the class/struct.


* It is important to remember there should only be one NSPersistentContainer that exists inside the app at any given time.

**Common Warning**: `Multiple NSEntityDescriptions Claim NSManagedObject Subclass ...`

Use [Unit Testing](#unit-testing) to make sure that you do not do this.
<br />




### Define Core Data model

* Define in your `.xcdatamodel` file
* Naming conventions:
	* Name your `xcdatamodel` file to be the same as your project name by default.
	* Name your entities to reflect what is being stored. Try to avoid suffixes like (-ing, -s, -ed, etc.)
		* Ex: For an entity that would store movies, make it singular as each record in this entity would be a single movie. So a good name for the entity would be `Movie`.
* Define all entites (Core data models) in the Core Data model file.
	* Attributes (properties)
	* Constraints
	* Relationships
<br />


### Managed Object

* An object that subclasses NSManagedObject can be called a **managed object**.
* By default Core Data will automatically generate the NSManagedObject subclasses that will allow you to use your entites in Swift similarly to a normal model.
	* The generated managed object will be named based on the name you gave your Entity. Ex:`Movie`  entity will generate a managed object called `Movie`.
	* By default the generated managed object classes will make your attributes optional in Swift
		* If you specified a Core Data entity like `Movie` to have an non-optional String attribute like `name` this would still be an optional String when you go to modify this record on the managed object.
		* If you give nil to managed object property like `movieInstance.name = nil` it will be converted to an empty string `""` to fit the configuration you specified in Core Data.
	* Generated managed objects will alway contain a static method called `fetchRequest()` that you can use to relatively easily create a fetch request for a managed object.
		* Note: If you try to call the fetchRequest method without first declaring its type like `let fetchRequest: NSFetchRequest<Movie> = Movie.fetchRequest()` then you will get an error to compile. 
			* There are two fetchRequest methods available for managed objects, one is `NSFetchRequest<NSFetchRequestResult>` and the other is your `NSFetchRequest<Movie>`
	* Manually subclass to avoid these weird little things.
* You HAVE to specify a value to non-optional attributes of the managed object. (duh). If you don't there will be an error thrown when saving the moc.
* Each managed object has an `objectID` that you cannot modify yourself. This can be passed for multithreading the same object. This property ensures that each managed object is unique and is used to reference a managed object throughout the entire stack. 
<br />


### Managed Object Context

* Need `NSManagedObjectContext` - the workspace that you can use to work with core data instances before saving them to the store (since that is an expensive process).
* Abbreviated Managed Object Context is "moc"
* Each managed object must be connected to a managed object context.
* All fetches from the Persistent Store will be loaded into the managed object context.
* Any changes (like updating or changing a managed object) that are made the managed object context must be saved in order to be persisted to the persistent store.
* Different instances of the same managed object can exist in different managed object contexts.
	* This is often a source of issues if you have a series of managed object contexts that are not properlly updating whenever one is saved.
* It is essential to manipulate managed objects on the thread that they were created on. So if you create an object that is on the main managed object context thread, then you must manipulate it on that thread or pass the id of that object to another background moc and create/manipulate it there.
* In case of errors when saving a context, it could be a good idea to call `rollback()` to discard all of the non-yet persisted changes. In other words, this returns your context to its empty state right after its last successful `save()`.
	* The reason to empty a context after an error is because if you have even one managed object in an error state, no other managed objects in the context will be able to save successfully.
```
do {
	try persistentContainer.viewContext.save()
} catch {
	persistentContainer.viewContext.rollback()
	print("Failed to save: \(error)")
}
```
* Whenever you are trying provide an easy way for the user to cancel their actions (like if they are editing a managed object but then they decide they don't actually want to save their edit), `rollback()` is potentially a good method to use. 
	* Beware this will discard **ALL** managed objects in the context.
<br />



### Adding new records

* Create add a instance of a managed object by initializing it with a managed object context:
```
let movie = Movie(context: persistentContainer.viewContext) // This is the default moc, main thread
```
* Make modifications on that object to give it the correct data:
```
movie.name = "The Great Ranger"
```
* Save the managed object context
```
do {
	try persistentContainer.viewContext.save() // call default save method on default moc
	print("Saved successfully")
} catch {
	print("Failed to save movie: \(error)")
}
```
<br />



### Retrieving records

* Powered by `NSFetchRequest`
* Fetch requests contain all information needed to retrieve the data that you are looking for in your Core Data store.
* Fetch requests are always executed by managed object contexts.
* Predicates (`NSPredicate`) are used for filtering.
* Sort descriptors (`NSSortDescriptor`) are used for (duh) sorting your results.

Basic fetch request to get all records of an entity:
```
extension StorageProvider {
	func getAllMovies() -> [Movie] {
		let fetchRequest: NSFetchRequest<Movie> = Movie.fetchRequest() // auto-generated method

		do {
			return try persistentContainer.viewContext.fetch(fetchRequest)
		} catch {
			print("Failed to fetch movies: \(error)")
			return []
		}
	}
}
```

* Placing the request in an extension of your storage provider like the example code above allows us to completely separate the Core Data logic from the UI logic. There are other ways this can be done to suit your architecture.
* Fetching data from core data does not automatically update your UI. There are ways to implement but they will have to be done in your view logic.
<br />




### Deleting records
```
extension StorageProvider {

	func deleteMovie(_ movie: Movie) {

		persistentContainer.viewContext.delete(movie)

		do {
			try persistentContainer.viewContext.save()
		} catch {
			persistentContainer.viewContext.rollback()
			// print error
		}
	}
}
```
<br />




### Modifying records
```
extension StorageProvider {

	func updateMovies() {

		do {
			try persistentContainer.viewContext.save()
		} catch {
			persistentContainer.viewContext.rollback()
			// print error
		}
	}
}
```
* Because managed objects are always connected to managed object contexts, whenever you update the managed object's property all that needs to be done to actually persist the change to the store to save the context.
<br />


### Multithreading
* Never use a persistent container's view context (main thread) for a fetch request in a network call's completion handler.
* Managed objects and their contexts are not thread-safe - Do not pass a managed object that was created in a main context to a background context.
<br />

Basic multithreading in iOS:
* All code in your app runs on a queue.
	* UI code must be run on the main queue.
	* Callbacks like for a network call will occur in *some* background queue (you don't know which).
* Grand Central Dispatch (GCD) manages the queues to make sure they get time to do their work.
<br />

Back to Core Data:
* Each managed object context is associated with some dispatch queue.
* You can create a new managed object context using your persistent container's `newBackgroundContext()` method:
```
let backgroundContext = persistentContainer.newBackgroundContext()
```
* When you create background context like this it will be directly connected the persistent store coordinator, which means that any saves on this context will save directly to the data store. 
	* If you want to update the main context, you must fetch these changes from the persistent store.
	* In this scenario the background context and main context would have a sibling-like relationship. Both can read and write from the persistent store coordinator.
* Background contexts should be used whenever you need to do heavy processing of managed objects or perform slow fetch requests. If performed on the main context it might block the UI.
* To use a managed object on a background context (as you must to make sure there are no threading issues), you should use the context's `perform` or `performAndWait` method.

Example to create a new managed object and save it to the persistent store from a background thread:
```
let backgroundContext = persistentContainer.newBackgroundContext()

backgroundContext.perform {

	let movie = Movie(context: backgroundContext)
	movie.name = "The Terminator"

	do {
		try backgroundContext.save()
	} catch {
		backgroundContext.rollback()
	}
}
```

* The `perform()` method is asynchronous which means:
	* Any changes made to the background context in the perform closure are not applied to the viewContext (main context) immediately.
	* All code in the perform block is run asynchronously which means it does not block the main queue.
* You can also use perform on the main context. This could be useful if you want to interact with the main context from a background queue, for example when you're in a network callback closure.
* To have a flexible asynchronous method to retrieve some Core Data information you could do something like this:
```
func fetchTodoItems(dueBefore: Date, in context: NSManagedObjectContext, _ completion: @escaping ([ToDoItem]) -> Void) {

	context.perform {

		let request: NSFetchRequest<ToDoItem> = ToDoItem.fetchRequest()
		request.predicate = NSPredicate(format: "dueDate < %@", dueBefore as NSDate)

		let items = try? context.fetch(request)

		completion(items ?? [])
	}
}
```
* This implementation works fine, but the thing is... most core data methods run synchronously so it is sometimes unnecessary to use completions like this (or Futures/Publishers if you're using Combine)
* A simpler alternative for methods like this that are syncrhonous is to use `performAndWait()` and return the managed objects fetched directly:
```
func fetchTodoItems(dueBefore: Date, in context: NSManagedObjectContext) -> [ToDoItem] {

	var items: [ToDoItem]?

	context.performAndWait {

		let request: NSFetchRequest<ToDoItem> = ToDoItem.fetchRequest()
		request.predicate = NSPredicate(format: "dueDate < %@", dueBefore as NSDate)

		items = try? context.fetch(request)
	}

	return items ?? []
}
```
* Because `performAndWait` runs synchronously, it will halt the queue until the closure passed to it completes.
* **NOTE**: `performAndWait` should only be used only if you have no alternative.
	* This method call has the potential to create deadlocks by having queues wait for each other to finish like any other blocking synchronous code.
	* Documentation states that the method is safe to be called reentrantly (you can call `performAndWait` from within a `performAndWait` closure without any problems)

* This example `performAndWait` and `perform` method has another small problem: We do not control what thread calls the method:
```
// BAD
DispatchQueue.global().async {

	// Background thread

	let viewContext = storageProvider.persistentContainer.viewContext

	// Fetches in main Context
	let items = fetchToDoItems(dueBefore: Date(), in: viewContext)

	for item in items {
		print(item.dueDate)
	}
}
```
* This above code can cause some very difficult to find EXC_BAD_ACCESS errors - It might compile most of the time but it might crash some day when a specific block of code is ran on a different thread than it did before.
	* To debug a little easier when yuo get this - open your scheme editor for your app target and pass the following launch argument to your app's **Run** scheme - `com.apple.CoreData.ConcurrencyDebug 1`
	* This will allow Core Data to perform strict checks to make sure you don't violate its threading rules.
	* This will crash your app if detects any violations.
	* You should see the following message in the console whenever your app launches:
```
CoreData annotation: Core Data multi-threading assertions enabled.
```
* It is not uncommon to run a fetch request on a background thread then display the results in the UI. For example, if you perform a heavy post-processing on a fetch request's results before displaying the outcome to the user.
	* To follow the Core Data threading rules, pass your managed object safely from one context to another

Passing an object safely from one context to another:
* Use the managed object's `objectID`. There is some information on in [Managed Object](#managed-object).
	* Because a managed object's `objectID` is persisted in the underlying store, it can be used to fetch managed objects directly.
	* This means that it is safe to fetch managed objects in one context and then use their IDs to fetch them in another context.

Ex:
```
DispatchQueue.global().async {

	// Background thread

	let viewContext = storageProvider.persistentContainer.viewContext
	let bgContext = storageProvider.persistentContainer.newBackgroundContext()

	// Fetches in main Context
	let items = fetchToDoItems(dueBefore: Date(), in: viewContext)

	// These are managed objects associated with the main context
	for item in items {

		bgContext.perform {

			// pull the managed object into the background context by using the object's id.
			let item = bgContext.object(with: fetchedItem.objectID) as? ToDoItem

			print(item.dueDate)
		}
	}
}
```
* Use `object(with:)` method to fetch a managed object from an ID.
	* If the object is already in the context that this method is being called in, no extra work will be done... because we already have the object that is associated with the context.
* If the object that you are looking for doesn't exist within the context yet, you'll be given a fault for the object.
	* Core Data assumes that whenever *faults* are fire the referenced managed object exists in the persistent store so it will search there.
	* Only use `object(with:)` when you are sure the object exists in the persistent store. If the object does not exist in the persistent store then your application will **CRASH**.
	* If you are not sure if the object you're looking for exists in the store, or if you just want to play it safe, then you'll want to use `existingObject(with:)`.
		* Does not return a fault if the object is not found in the current context. Instead, it uses a throwing method to search the persistent store.
		* If the persistent store does not have the object, it will throw an error that you can handle however you want.
		* Prefer this method over `object(with:)` cause who doesn't love safety?
* Every time that you use `existingObject(with:)` or `object(with:)` to pull a managed object into a different thread you are requiring a roundtrip to the persistent store. Therefore, this should be used as sparingly as possible - only when you need to access the properties of a managed object in another thread.
	* If you are not sure if you want to actually use the managed object's properties, then you could get away with simply passing the managed object ID to the other context and only return the managed object (associated with the new context) when it is necessary to access its properties.
	* Here's a small abstraction that gets objects in a background context and then retrieves them in a main context when it is time to actually show them in some List or CollectionView:
**TODO**
```
class ToDoItemDataSource {

	private var fetchedIDs: [NSManagedObjectID] = []
	private var objects: [NSManagedObjectID: ToDoItem] = [:]
	private let persistentContainer: NSPersistentContainer

	var numberOfItems: Int { fetchedIDs.count }


	init(persistentContainer: NSPersistentContainer) {

		self.persistentContainer = persistentContainer
	}


	func fetch(_ completion: @escaping () -> Void) {

		// Since you do not need to keep a reference to the newBackgroundContext(), you could instead
		// use persistentContainer.performBackgroundContext(_:) to directly run this fetch on a
		// background context.

		let context = persistentContainer.newBackgroundContext()
		
		context.perform { [weak self] in

			// Since you just want the IDs in this case, you could get away with just fetching those
			let request: NSFetchRequest<NSManagedObjectID> = NSFetchRequest(entityName: "ToDoItem")
			request.resultType = .managedObjectIDResultType // only returns the NSManagedObjectID

			/*
			If you wanted to additionally sort the objects (by date for example) add the following to
			your request:

			request.sortDescriptors = [NSSortDescriptor(key: "dueDate", ascending: true)]
			request.propertiesToFetch = ["dueDate"]
			*/


			self?.fetchedIDs = (try? context.fetch(request)) ?? []

			completion()
		}
	}


	func object(at index: Int) -> ToDoItem? {

		let id = fetchedIDs[index]
		if let object = objects[id] {
			return object
		}
		let viewContext = persistentContainer.viewContext
		if let object = try? viewContext.existingObject(with: id) as?
		ToDoItem { ,!
		objects[id] = object
		return object
		}
		return nil
	}
}
```
How the above code works:
* You would call `fetch(_:)` on some background thread whenever you want to retrieve all of the objects from the persistent store. 
* The fetch provides a completion handler to notify you whenever the method finishes.
* Whenever you are ready to present to the UI through a SwiftUI `List` or a UIKit `TableView`, you can call `object(at:)` and load in the objects stored the data source when needed.
	* If an objectID has been fetched from the persistent store already, it will be stored in a dictionary (`objects`) so that we fetch from there instead of going to disk again.
	* If an objectID has not been fetched before, we will unfortunately have to go to disk to find it then we will cache it, in case it is used later.
* This kind of abstraction would be most useful whenever we are fetching many many objects from the persistent store and we do not want go through the process of loading all of them in main context. OR if you are doing significant post processing on the objects that you are fetching. Otherwise, DON'T USE THIS.
* **NOTE:** You cannot call the `object(at:)` method from the background context. It kind of completely defeats the purpose of this datasource because you would be handed back a main context associated object which would cause the same threading issues as normal. Luckily this should be caught if you have concurrency debugging enabled (as shown earlier) so it should crash and instantly let you know this was a bad idea.
* Generally, the above is not going to be necessary to do often. In most cases you should be able to work with the managed objects directly in the thread that they were called in.
<br />


* If you ever need to use a temporary background context to save directly to your persistent coordinator for expensive operations, like you could in the class above, `persistentContainer.performBackgroundContext(_:)` is your guy/gal.
<br />


#### Child contexts
**TODO**
* This is another way to create and use temporary managed object contexts that is more appropriate for UI operations.
* A child context is a managed object context that is a direct child of another managed object context. When you save a child context, these changes are not persisted to the persistent store but instead to its parent context.
* It is up to the parent context to save to the persistent store (or the next parent if it is a nested child context).
* Child contexts read data through its parent, but it might end up receiving data from the persistent store depending on whether or not the parent context can provide data from its row cache.
* Useful for when you pass a managed object from one view to another view that allows the user to make modifications to the managed object.
	* If the managed object is attached to a child managed object context then you can safely make changes to the managed object without dirtying your main context.
	* If the user cancels the modifications that they were making on the object then you can easily discard the child managed object context and the user's modifications are discarded with it.
	* If the user wants to save the modifications that they were making then you can call `save()` on the child context and it will go straight to the main context. Then call `save` on the main context to persist to the store.
Ex:
```
class EditViewController: UIViewController {

	let object: ToDoItem
	let context: NSManagedObjectContext?
	let parent: NSManagedObjectContext // must have a parent


	init(sourceObject: ToDoItem) {

		guard let parentContext = sourceObject.managedObjectContext else {

			fatalError("Attempting to edit a managed object that is not associated with a context")
		}

		self.parentContext = parentContext

		// New stuff!
		let childContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
		childContext.parent = parentContext

		guard let childObject = childContext.existingObject(with: sourceObject) else {

			// We could not find the item in the childContext. Recover by using the 
			// object's context instead.

			// This is not ideal and should never happen but it is nicer than fatalError.

			self.context = nil
			self.object = sourceObject
			return
		}

		// Child context created, ToDoItem fetched
		self.context = childContext
		self.object = childObject
	}


	func persistChanges() {

		defer {
			// dismiss this vc when we return
			presentingViewController?.dismiss(animated: true, completion: nil)
		}

		guard context?.hasChanges else {
			return // no changes to persist
		}

		do {
			try context?.save()
		} catch {
			print("Something went wrong saving the child context: \(error)")
			return // don't save the parent
		}

		do {
			try parentContext.save()
		} catch {
			print("Something went wrong while saving the parent context: \(error)")
		}
	}
}
```
* While you don't *have* to save your parent context after saving the child context, you need to be aware that your changes from the child context will not be saved in the parent context until after you have saved the parent.
* Child contexts are the preferred tool to isolate any changes that the user makes to managed objects in the UI due to their saving behavior.
* When the child context is discarded, when this view controller is deinitialized, all of the managed objects owned by that child are discarded as well.
<br />


#### Responding to changes in another managed object context
* Working with multiple contexts will often involve responding to changes that were made in one context to update another context.

* There are several reasons that you would want to know when a context changes:
	* To pull changes that were made in the changed context into another context.
	* To reload your UI (maybe you want to do this when a specific context is updated).
	* To run some logic when any context updates.

* Core Data has a mechanism to allow you to benotified when a managed object updates.
	* This mechanism is key in objects like `NSFetchedResultsController` which tracks a specific managed object context to figure out whether objects were inserted, deleted, or updated.
		* Also tracks whether the position of a managed object within a result set has changed, which is not easy to do yourself.
* Observe and respond to changes in your managed object contexts through `NotificationCenter`. When your managed object context updates or saves, Core Data will post a notification to the default `NotificationCenter` instance.
Ex:
```
class ExampleViewModel {

	init() {

		let didSaveNotification = NSManagedObjectContext.didSaveObjectsNotification

		NotificationCenter.default.addObserver(
			self, 
			selector: #selector(didSave(_:)), 
			name: didSaveNotification,
			object: nil
		)
	}


	@objc func didSave(_ notification: Notification) {

		// handle the save notification
	}
}
```
* The above will notify you whenever any managed object context is saved.
* The notification you receive will contain a `userInfo` dictionary which will tell you what objects were inserted, deleted, and/or updated.
Ex:
```
@objc func didSave(_ notification: Notification) {

	// handle the save notification
	let insertedObjectsKey = NSManagedObjectContext.NotificationKey.insertedObjects.rawValue
	print(notification.userInfo?[insertedObjectsKey])
}
```
* The above snippet will extract inserted objects from the `userInfo` dictionary using a nested type called `NotificationKey`.
* Since the enum case for the notification keys don't match with the string that you need to access the relevant key in the dictionary, it's important to use the enum's rawValue. 
* **NOTE**: This is only for iOS 14+
	* For iOS 13 and below: use `Notification.Name.NSManagedObjectContextDidSave` to monitor the save events.
<br />


Useful extensions to make this process little cleaner:
* Extensions can be used on `Dictionary` to help save some typing:
```
extension Dictionary where Key == AnyHashable {

	func value<T>(for key: NSManagedObjectContext.NotificationKey) -> T? {

		return self[key.rawValue] as? T
	}
}

// Now the didSave function can be made to look like:

@objc func didSave(_ notification: Notification) {

	// handle the save notification
	let inserted: Set<NSManagedObject>? = notification.userInfo?.value(for: .insertedObjects)
	print(inserted)
}
```
* But we can go even further! Create an extension for Notification to so that we can make a call just for inserted objects and get given the objects without parsing through the `userInfo` dictionary at the call-site
```
extension Notification {

	var insertedObjects: Set<NSManagedObject>? {
		return userInfo?.value(for: .insertedObjects)
	}
}

// Now the didSave function can be made to look like:

@objc func didSave(_ notification: Notification) {
	let inserted = notification.insertedObjects
	print(inserted)
}
```
* The only problem with this method of extracting logic from the callsite is that you will need to manually create the computed properties for each of the different values that the notification's `userInfo` data could have.
	* For example you will need to make another computed variable for `.updatedObjects` and `.deletedObjects` if you are trying to monitor them.
<br />


Back to observing managed object contexts:
* The previous example of subscribing to `NSManagedObjectContext's` `.didSaveObjectsNotification` will trigger every time any managed object context saves.
* You can specify the managed object context that you want to monitor, instead of listening for *ANY* of them to save:
```
let didSaveNotification = NSManagedObjectContext.didSaveObjectsNotification
let targetContext = persistentContainer.viewContext

NotificationCenter.default.addObserver(
	self, 
	selector: #selector(didSave(_:)), 
	name: didSaveNotification,
	object: targetContext
)
```
* When you pass a reference to a managed object context to addObserver, you can make sure you're only notified when a specific managed object context was saved.

* Let's imagine that you have two contexts, a background context and a main context. Whenever your background context saves, you would want to update your main context so that your UI is up to date. 

* If you subscribe to all managed object context did save notifications and simply update your UI when any context got saved, you could do this.
	* This requires that you have set `automaticallyMergesChangesFromParent` on the main context to true. This will make your context automatically pull in changes from its parent (the persistent container) whenever it is changed.
	* So ideally you would monitor for a save, then reload your UI to reflect the new data. (Or it would simply reload automatically when the data changes)

* `automaticallyMergesChangesFromParent` is set to false for `viewContext` by default. So it will not typically update whenever you save a background context unless you tell the `viewContext` to pull the changes that were made by the background context.

* You can make manually sure that a managed object context merges changes from another managed object context by subscribing to the `didSaveObjectsNotification` and merging in any changes contained in the notification:
```
@objc func didSave(_ notification: Notification) {

	persistentContainer.viewContext.mergeChanges(fromContextDidSave: notification)
}
```

* Calling `mergeChanges` on a managed object context will automatically refresh any managed objects that have changed.
* This ensures that your context always contains the latest information.
* **Note**: Like stated previously, `mergeChanges()` is a manual way to keep contexts up to date. If you have set your context to automatically merge changes from parent, Core Data will do this for you automatically.
* Setting `automaticallyMergesChangesFromParent` to true is not always the best approach
**TODO**
* For example, if you are importing a large amount of data and incrementally saving the context, you might be refreshing your UI with new objects several times before the import is completed.
	* In this case you would instead want to explicitly refresh your main context after the import is complete.
	* Do this by calling `reset()` on the main context and then fetching your data again.
	* `reset()` will empty the managed object context of all objects in memory. You will need to fetch data from the persistent store to reload your UI.
* If you would like to be notified whenever a managed object context was changed then you can observe `didChangeObjectsNotification`.
	* Maybe a time where this would be nice is after `mergesChanges` is called so that you can tell what you just merged into your context.
* The `userInfo` dictionary in the `Notification` object that is returned from this subscription allows you to access the changed objects as long as you access them from the context that generated the notification.
* **NOTE**: Batch inserts, deletes, and updates will not trigger these notifications. 

<br />

#### Query genertions
* A problem that often occurs whenever you are working with multiple contexts is that an instance of an entity in one context can be different than an instance of the same record in another context.

* More importantly, one managed object context can hold a fault that points to a record that was deleted by another managed object context.

* This can be mitigated by using `automaticallyMergesChangesFromParent` to true for all of the contexts however there are trade-offs to this as mentioned above, specifically in creating an overhead of many trips to the data store if you are making many saves for some reason.

* However, this would not solve the problem in all cases. If one context has fetched an object as a fault and another context deletes the same object before the fault is fulfilled.

* A solution: Set `shouldDeleteInaccessibleFaults` on your managed object context to `true`. This makes sure that your managed object context knows that an object that no longer exists in the persistent store can be accessed.
	* When you access a deleted object Core Data will fail silently and you will have to handle this yourself.

* A better solution: Query generations allows you to prevent problems like this altogether, though.
* **Query generations** allows you to pin a managed object context to a specific snapshot of the underlying store.
	* Faults will be fulfilled using your snapshot of your persistent store rather than the actual persistent store.
	* Any fetch requests that you execute after pinning your managed object context to a query generation will be run against a snapshot of the persistent store rather than the persistent store itself.
	* By default, managed object contexts always point to the latest query generation.
* A new query generation is created every time the persistent store is changed via a managed object context saving its managed objects.
* This is desired behavior if you are only ever using one moc.
* If you are creating a bunch of changes in a background managed object context then you might want to control want to be safe about what the viewContext reads from the persistent store. You can do this by managing the snapshot that the viewContext works on. 
	* I am unclear how this is different than simply not updating your stuff on the main queue whenever something changes the persistent store. Maybe it is because you could have a fault even when you are not updating the main context with the persistent store that causes it to look into the persistent store for some fetch...
* An example of a real-life situation where this could be useful is if want to be able to use a persistent store to filter, update, or simply use the UI in whatever way you want while you are loading paginated data from a remote source and adding it to the persistent store in increments. This way you will not absorb the new changes in the database until it is all done.
**TODO**
* To pin a managed object context use `setQueryGenerationFrom(_:)` on the managed object context using the query generation token that you want to pin the managed object context to.
* Get a query generation token for a managed object context's current query generation use `queryGenerationToken`.
* So lets say we want to pin the main context before we start a fetch request:
```
let currentToken = persistentContainer.viewContext.queryGenerationToken

try? persistentContainer.viewContext.setQueryGenerationFrom(currentToken)

// perform fetch request.
```
* The above would make the query use the snapshot corresponding to the current context and any faults that occur would operate on this snapshot.
* At some point you would need to set your viewContext to the latest query generation again. Like when you finished downloading all of the data to your data store - or whatever you were doing. Or if the user taps a 'refresh' button.
* Get the most recent query generation token by accessing the `current` class var on `NSQueryGenerationToken`:
```
let currentToken = NSQueryGenerationToken.current

try? persistentContainer.viewContext.setQueryGenerationFrom(currentToken)

```
* After you set the current snapshot you will still need to update the managed object context's managed objects to reflect this new snapshot. Call `refreshAllObjects()` to do this.

* Another way to reset your managed object context to the newest query generation is to call `reset` on the context and return it to the base state. This will automatically be pinned again to the current query generation. You will need to re-execute your fetch requests from the persistent store again.
	* **NOTE**: This will make your managed object context stay at the query generation at the time of the `reset` instead of automatically pinning to a new generation. (This is only whenever you have pinned a query generation in the first place) - If you don't pin, `reset` will behave by always pinning to the latest snapshot automatically.

* Alternatively, You can update a managed object context by calling `save()` to push some changes. After doing this it should automatically pin to the latest from here on.

* Lastly, update to the latest query generation when it merges changes from another managed object context. Which, of course means that you can't use `automaticallyMergesChangesFromParent` to true on the context that you want to set to a specific query generation.

* This would be logic that you would want to put in something like a completion handler of a really complicated core data import. So that you could update on it finishing.



### Unit testing

#### Setting up an in-memory SQLite store
* When you want to unit test an object that depends on Core Data or use Core Data in your unit tests, always create a temporary persistent container that does not share its storage with other containers.
* While Core Data supports an in-memory store type, we're not going to use it. However, we ARE going to use an in-memory store.
* The idea behind in-memory store:
	* Normally, a persistent container set up with an SQLite store writes to a file on disk.
	* Every persistent container that you create with the same configuration will persist data to the same underlying SQLite file. 
	* Not great for unit tests since
		* we don't want the store to outlive the test,
		* you would never be able to run multiple unit tests in parallel because they would read and write from the same store, and when one tests runs cleanup code another test might still be running.
	* Instead, with an in-memory store, the persistent container that you create would use a unique space in memory that means that containers would not share their underlying storage.
	* This allows multiple containers to exist in your app and operate independently.
	* This is never persisted to disk so there is no need to perform clean-up.

* Why not use Core Data's default in-memory store?
	* Some Core Data features only work in an SQLite store.
		* Some predicates only work when you're not using an SQLite store. Only features like certain delete rules only work in an SQLite store.
	* This can lead to inaccuracy in your tests versus actual application results.

* To set up an SQLite store that operates on an in-memory store set the path for your SQLite file in your persistent container to `/dev/null`.

Ex:

```
class StorageProvider {

	let persistentContainer: NSPersistentContainer

	init(storeType: StoreType = .persisted) {

		persistentContainer = NSPersistentContainer(name: "whateverItIs")

		if storeType == .inMemory {

			persistentContainer.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
		}

		persistentContainer.loadPersistentStores(completionHandler: { description, error in

			if let error = error {
				fatalError("Core Data store failed to load with error: \(error)")
			}
		})	
	}
}


enum StoryType {

	case inMemory, persisted
}
```

* Now in a test you could create a `StorageProvider` that is in-memory like:
```
class Chapter11Test: XCTestCase {

	var storageProvider: StorageProvider!

	override func setUpWithError() throws {

		storageProvider = StorageProvider(storeType: .inMemory)
	}


	func test_AddItem_PersistsToDoItem() throws {
		let request: NSFetchRequest<ToDoItem> = ToDoItem.fetchRequest()

		let context = storageProvider.persistentContainer.viewContext
		let initialCount = try context.count(for: request)
		XCTAssertEqual(initialCount, 0)

		storageProvider.addToDoItem(title: "Test item", dueDate: nil, in: context)

		let finalCount = try context.count(for: request)
		XCTAssertEqual(finalCount, 1)
	}
}
```

