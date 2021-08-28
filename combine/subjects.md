# Subjects

A **subject** in Combine is a special kind of publisher that allows you to inject values into its stream. 

Subjects in Combine have a `send(_:)` method that allows you to do this. It is also possible to use `send(completion:)` to complete a stream of values (and end the publisher, I believe) if needed.

* [PassthroughSubject](#passthroughsubject)
* [CurrentValueSubject](#currentvaluesubject)
* [Published](#published)
* [Practical applications](#practical-applications)

## PassthroughSubject
Sends a stream of values from their origin, through the publisher, to its subscribers. The origin of the value stream is often a stateless, imperative code. 

* This subject is good for publishing ephemeral values, like events.
	* For example, in agame, you could use a PassthroughSubject to communicate that the player has retrieved an item or completed a level. The subscriber would then update the state of the program/handle the event.




## CurrentValueSubject
Very similar to a PassthroughSubject, however different in that instead of passing something to the subscribers they are getting the current value of some subject (kind of as the name implies).

Example from Practical Combine:
```
class Car {
	var kwhInBattery = CurrentValueSubject<Double, Never>(50.0)
	let kwhPerKilometer = 0.14

	func drive(kilometers: Double) {
		let kwhNeeded = kilometers * kwhPerKilometer
		assert(kwhNeeded <= kwhInBattery.value, "Can't make trip, not enough charge in battery")
		kwhInBattery.value -= kwhNeeded
	}
}

let car = Car()

// don't forget to store the AnyCancellable if you're using this in a real app

car.kwhInBattery
	.sink(receiveValue: { newCharge in
		someLabel.text = "The car now has \(newCharge)kwh in its battery"
	})

// If you call the drive function
car.drive(kilometers: 100) // label will now automatically update to show the new amount of wats left in the battery.
```

The key thing to release in this example is that we are automatically going to be calling the label to update whenever the kwhInBattery is changed anywhere in the Car class.
* Note: You need to access the value in the subject by calling `.value` so that you can access the actually variable and not the subject itself.


## Published
This `@Published` property wrapper is essentially an equivalent to CurrentValueSubject with a limitation that it can only be used with classes. I don't understand all of the intricacies yet, but if I ever do, here is a blog [source](https://www.donnywals.com/publishing-property-changes-in-combine/), or I'm sure further detail is in the Practical Combine book.

## Practical applications
### Abstracting NSFetchedResultsController
Something useful that I've found in the Practical Core Data book (by Donny Wals) is a little class called `MovieProvider` (or whatever entity). Inside is an abstraction to use NSFetchedResultsController. 

#### The initial setup. 
In the Donny Wals guide, I get the sense that he is purposely trying to not use any architecture terms so that there less of a learning curve to pick on the things that he is trying to get across. Unfortunately. I want to put his code in an MVVM architecture, and instead of that, he has this `MovieProvider` that acts as an `NSFetchedResultsController` directly connected to his View like so:

* The `MovieProvider` is an `ObservableObject`.
* It would manually call `objectWillChange.send()` whenever it detected a change to the fetched results controller. 
* That `objectWillChange` property was possible since `MovieProvider` was an `ObservableObject` 
	* If you connected it directly to your SwiftUI view using a `@ObservedObject` property wrapper whenever the observed object notifies that it changed by sending `objectWillChange.send()`, the view knows to update accordingly.


Initial setup code `MovieProvider`:
```
//
//  MovieProvider.swift
//  CoreDataDemo1
//
//  Created by Boyce Estes on 4/28/21.
//

import Foundation
import CoreData
import Combine

/*
 * This holds all logic responsible for the NSFetchedResultsController.
 * Whenever we break it into its own class like this we can separate the
 * Core Data logic from the rest of the business logic in the view-model.
 *
 * It makes for a cleaner view-model and allows for us to move class with
 * the rest of the Core Data classes.
 */
class MovieProvider: NSObject, ObservableObject {

    // MARK: - Properties
    fileprivate let fetchedResultsController: NSFetchedResultsController<Movie>
    
    var numberOfSections: Int {
        return fetchedResultsController.sections?.count ?? 0
    }


    // MARK: - Lifecycle
    init(storageProvider: StorageProvider3) {
        
        let fetchRequest = Movie.createFetchRequest()

        // All movies
        fetchRequest.sortDescriptors = [NSSortDescriptor(keyPath: \Movie.title, ascending: true)]
        self.fetchedResultsController = NSFetchedResultsController(
            fetchRequest: fetchRequest,
            managedObjectContext: storageProvider.mainContext,
            sectionNameKeyPath: nil,
            cacheName: nil
        )
        super.init()
        
        fetchedResultsController.delegate = self
        try! fetchedResultsController.performFetch()
    }
    

    // MARK: - Methods
    func numberOfItemsInSection(_ section: Int) -> Int {
        
        guard let sections = fetchedResultsController.sections,
              sections.endIndex > section else {
            return 0
        }
        
        return sections[section].numberOfObjects
    }
    
    
    func object(at indexPath: IndexPath) -> Movie {
        
        return fetchedResultsController.object(at: indexPath)
    }
}


// MARK: - Fetched Results Controller Delegate
extension MovieProvider: NSFetchedResultsControllerDelegate {
    
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {

    	objectWillChange.send()
    }
}

```

#### Introducing a view-model
As said above, instead of connecting the data directly to the table, I want everything to go through the view-model instead. The question now is, "How can I get the MovieProvider class to update the view-model which should then update the view?". We can't use ObservedObjects because that only works for updating Views. Or at least I believe that it is good practice to only use them like that.

The answer is custom subjects (manual publishers (I like to call them)). 

I decided to go with a [PassthroughSubject](#passthroughsubject) since I don't actually need any information passed (that's all going to be in the `MovieProvider`), I just need to let the view-model know that it has changed. From there the view-model will be our new `ObservableObject` that should call `objectWillChange.send()` to the trigger the change in the View.

This is simpler than you would think. We just need to make sure that we create the subject where we will need to emit values. In this case, it doesn't really matter what value we pass in the subject. What matters is that we pass a value at the correct time. I just set the value to: `MovieProvider`

```
let moviePassthroughSubject = PassthroughSubject<MovieProvider, Never> // Never has errors.
```

After this subject is initialized at the beginning of the file, we can easily replace `objectWillChange.send()`, which gets called whenever there are changes in the fetched results controller, to `moviePassthroughSubject.send(self)` .

In the view-model we can reconcile these changes and call the View to update by subscribing to this subject and telling the View that it needs to update whenever the subject emits a value.

```
class MovieViewModel: ObservableObject {

    // MARK: - Properties
    var movieProvider: MovieProvider

    var cancellables = Set<AnyCancellable>()

    // MARK: - Lifecycle
    init(storageProvider: StorageProvider3) {
        
        movieProvider = MovieProvider(storageProvider: storageProvider)

        bindMovieProvider()
    }


    func bindMovieProvider() {

        movieProvider.moviePassthroughSubject.sink { [weak self] _ in
            print("Changed!")
            self?.objectWillChange.send()
        }.store(in: &cancellables)
    }


    // MARK: - Methods
    func numberOfItemsInSection(_ section: Int) -> Int {

        return movieProvider.numberOfItemsInSection(section)
    }


    func object(at indexPath: IndexPath) -> Movie {

        return movieProvider.object(at: indexPath)
    }


}
```

Whenever something happens like destroying and recreating the NSFetchedResultsController, the subject for the movie provider is not triggered... although I could if I wanted to for some reason.

#### Further reflection
I am not completely sure that this is the most optimized way to do this. It is possible that it might be a better idea to handle most of the updating using a CurrentValueSubject.

We would be able to subscribe to the current value and whenever we have a change... no that wouldn't work since we can't spoonfeed the data to the View very easily. It is probably a better idea to just have it get the data for each row like it currently is by caling `object(at:)` .

So I'm happy with this solution for now.

#### Even further reflection
It's important to note that the binding only gets changes passed when the fetchedr esults controller changes right now. You would need to manually send from the `moviePassthroughSubject` whenever you would want to do much else.
