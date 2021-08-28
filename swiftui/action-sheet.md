# Action sheet

First we need some sort of mutable variable, should pretty much always be private.
```
@State private var showFilterSheet = false)
```

Inside `var body: some View`:
```
.actionSheet(isPresented: $showingActionSheet) {
    ActionSheet(title: Text("Change background"), message: Text("Select a new color"), buttons: [
        .default(Text("Red")) { self.backgroundColor = .red },
        .default(Text("Green")) { self.backgroundColor = .green },
        .default(Text("Blue")) { self.backgroundColor = .blue },
        .cancel()
    ])
}
```


Source: https://www.hackingwithswift.com/quick-start/swiftui/how-to-show-an-action-sheet