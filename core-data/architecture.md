# Architecture

3-May-2021
This is kind of stumping me in terms of the "best" architecture to interact with Core Data. It would be nice to keep all of the logic for Core Data in a separate layer that would give clean results back to whatever view-model calls for it.

That being said, there are a few things like NSFetchedResultsController and fetch requests where I don't know where it should be stored in that context. 

* Create a NSPersistentContainer wrapper
	* Add useful functions like `save(backgroundContext:)` here.
	* Add way to create an in-memory version of the sqlite database that can be used in tests.
		* One way: Subclass the NSPersistentContainer wrapper and reset the lazy var `persistentContainer` to use an in-memory file path in in the subclass.
		* Alternatively: Have a parameter in the initializer that will denote whether the NSPersistentContainer wrapper that you create will be in-memory or not. Create it there. This loses the laziness, so it might make your startup time a little longer, but it is a little simpler for implementing the one-model loading.
	* Create logic to only load one model in here. More information in [Common errors](common-errors)

Other than this, it is a free-for-all. This document will need to be updated whenever I have a better idea of what works and what doesn't work. Until then. Wild-wild-west.

Oh. But I'm going to try to put fetches in the managed object subclasses.

And things that all of the managed objects use in the PersistentContainer wrapper. (like `save()`)