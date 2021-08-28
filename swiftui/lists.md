* [Lists](#lists)
* [ForEach](#forEach)
* [Sections](#sections)
* [Collapsible](#collapsible)
    * [Lists](#collapsible-lists)
    * [Sections](#collapsible-sections)


## Lists


## ForEach

When you want to loop over a sequence to create views, theres a strong possibility you want to use `ForEach`. 

The biggest distinction between this and a normal `forEach()` method in Swift, is that this is a bonified View in its own right. This means you can directly return it from your view body if needed.

Ex:
This will create 10 views in descending order. You would use each number in the range as the identifier for the items. This is possible because they are each going to be unique.

```
VStack(alignment: .leading) {
    ForEach((1...10).reversed(), id: \.self) {
        Text("\($0)…")
    }

    Text("Ready or not, here I come!")
}
```

[More info](https://www.hackingwithswift.com/quick-start/swiftui/how-to-create-views-in-a-loop-using-foreach)


## Sections
You can create a section to denote areas in a list using the `Section` View.

```
List {
    Section(header: Text("My Head"), footer: Text("My Foot")) {
        Text("Billy")
        Text("Mandy")
        Text("Grim")
    }
}
```

## Collapsible 
Since iOS 14 (Swift 2.0(ish)), there was introduced some new objects and parameters to create collapsible and expandable Lists.

### Collapsible lists
The simplest way is by creating a list using the new `children` constructor paramter. This parameter takes a keypath to a group of items that is the same format as the root. For instance, this is the example model used in WWDC '20.

```
List(activeMembers, children: \.children) { member in
    Text(member.name)
}
```

In this case, `activeMembers` is an array of Member objects. The Member object *must* have an optional array of the same type, in this case it is called `children`. This is what the KeyPath must point to.

This is the model that was used in the above example:
```
struct Member: Identifiable {
    let id = UUID()
    let name: String
    let children: [Member]?

    init(name: String, children: [Member]? = nil) {
        self.name = name
        self.children = children
    }
}
```

### Collapsible Sections


### Sources:
* https://developer.apple.com/videos/play/wwdc2020/10031/
