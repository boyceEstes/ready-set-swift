* [Lists](#lists)
* [ForEach](#forEach)


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