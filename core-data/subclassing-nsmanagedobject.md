
* [Creating NSManaged properties](#creating-nsmanaged-properties)
* [Common errors](#common-errors)


# Subclassing NSManagedObject

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


## Common Errors

#### Property cannot be marked @NSManaged because its type cannot be represented in Objective-C

This is mentioned in [Creating NSManaged properties](#creating-nsmanaged-properties) but the reason this is happening is because `NSManagedObject` utilizes runtime features of Objective-C. This means that all the attributes of the subclass must be representable in Objective-C.

If you want an attribute Optional, you may ned to change the type from something like `Double` to `Decimal` or `NSNumber`.

`Set<GenericType>` values are still okay to use optionally.