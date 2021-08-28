# Navigation
* [Navigation bar](#navigation-bar)
	* [Creating a ToolbarItem](#creating-a-toolbaritem)
* [Navigation view](#navigationview)
	* [Create inline navigation bar](#create-inline-navigation-bar)
* [Modals](#modals)

## Navigation bar
In SwiftUI, creating a navigation bar is a little bit different.

There are a couple of ways to do it, look through Evernote to complete this documentation. 

Basically `.toolbar(content:)` is used so that we can have a consistent navigation bar even when we are programming for WatchOS.

### Creating a ToolbarItem
```
.toolbar(content: {
    ToolbarItem(placement: .navigationBarLeading) {
        Button(action: {
            print("filter!")
        }, label: {
            Image(systemName: "arrow.up.arrow.down")
        })
    }
})
```

#### Source: https://www.hackingwithswift.com/quick-start/swiftui/how-to-add-bar-items-to-a-navigation-view


### Create inline navigation bar
```
NavigationView {
    Text("Hello, SwiftUI!")
        .navigationBarTitle("My nav title!", displayMode: .inline)
}
```

Alternatively,
```
NavigationView {
    Text("Hello, SwiftUI!")
        .navigationBarTitleDisplayMode(.inline)
}
```


## Navigation view
This is required whenever you are performing navigation in SwiftUI
```
struct SecondView: View {
    var body: some View {
        Text("This is the detail view")
    }
}

struct ContentView: View {
    var body: some View {
        NavigationView {
            VStack {
                NavigationLink(destination: SecondView()) {
                    Text("Show Detail View")
                }
                .navigationTitle("Navigation")
            }
        }
    }
}
```

### Source:
https://www.hackingwithswift.com/quick-start/swiftui/how-to-push-a-new-view-onto-a-navigationview


### Color Navigation Bar
```
//
//  NavigationBarModifier.swift
//  LiftMe
//
//  Created by Boyce Estes on 5/23/21.
//

import SwiftUI

struct NavigationBarModifier: ViewModifier {

    var backgroundColor: UIColor?
    var titleColor: UIColor?

    init(backgroundColor: UIColor?, titleColor: UIColor?) {
        self.backgroundColor = backgroundColor
        let coloredAppearance = UINavigationBarAppearance()
        coloredAppearance.configureWithTransparentBackground()
        coloredAppearance.backgroundColor = backgroundColor
        coloredAppearance.titleTextAttributes = [.foregroundColor: titleColor ?? .white]
        coloredAppearance.largeTitleTextAttributes = [.foregroundColor: titleColor ?? .white]

        UINavigationBar.appearance().standardAppearance = coloredAppearance
        UINavigationBar.appearance().compactAppearance = coloredAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = coloredAppearance
    }

    func body(content: Content) -> some View {
        ZStack{
            content
            VStack {
                GeometryReader { geometry in
                    Color(self.backgroundColor ?? .clear)
                        .frame(height: geometry.safeAreaInsets.top)
                        .edgesIgnoringSafeArea(.top)
                    Spacer()
                }
            }
        }
    }
}

extension View {

    func navigationBarColor(backgroundColor: UIColor?, titleColor: UIColor?) -> some View {
        self.modifier(NavigationBarModifier(backgroundColor: backgroundColor, titleColor: titleColor))
    }

}

```

Call with:
```
var body: some View {
	NavigationView {
		Text("This is the content!")
		.navigationBarTitle("Home", displayMode: .inline)
    	.navigationBarColor(backgroundColor: .systemBlue, titleColor: .white)
	}

}			
```

## Modals
In SwiftUI, modals are also called Sheets. You can display these in the View by calling `.sheet(isPresented: <Some-binding>)`

```
struct ContentView: View {
    @State private var showingSheet = false

    var body: some View {
        Button("Show Sheet") {
            showingSheet.toggle()
        }
        .sheet(isPresented: $showingSheet) {
            SheetView()
        }
    }
}
```