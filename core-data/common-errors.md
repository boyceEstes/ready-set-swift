# Common errors

* [Property cannot be marked @NSManaged because its type cannot be represented in Objective-C](#property-cannot-be-marked-nsmanaged-because-its-type-cannot-be-represented-in-objective-c)

### Property cannot be marked @NSManaged because its type cannot be represented in Objective-C

This is mentioned in [Creating NSManaged properties](./subclassing-nsmanagedobject.md#creating-nsmanaged-properties) but the reason this is happening is because `NSManagedObject` utilizes runtime features of Objective-C. This means that all the attributes of the subclass must be representable in Objective-C.

If you want an attribute Optional, you may ned to change the type from something like `Double` to `Decimal` or `NSNumber`.

`Set<GenericType>` values are still okay to use optionally.
