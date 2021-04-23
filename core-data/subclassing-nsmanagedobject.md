# Subclassing NSManagedObject

* [Creating NSManaged properties](#creating-nsmanaged-properties)


## Creating NSManaged properties

You can create a property to reflect your Core Data model by using the `@NSManaged` property wrapper before your variable declaration. 

Ex:
```
@NSManaged public var dateTimeLastUpdated: Date
@NSManaged public var dateTimeCreated: Date
@NSManaged public var name: String
@NSManaged public var exercises: Set<ExerciseTemplate>?
```

Because Core Data runs using Objective-C runtime features, the properties marked `@NSManaged` must be representable in Objective-C. 

This means that we are more limited in our use of data types than usual. Specifically in terms of Optionals. Even though we can have an optional Double in the Core Data model, we cannot in the subclass. They are inherently different things