# Navigation After New Entry

> The ,sheet modifier in stationListView has an onDismiss argument that is optional, but can be added int.  It is a closure that can performa an action when the presented view is dismissed.
>
> What we want to do is to check to see if a new Station was created (meaning the cancel button was not tapped) and inserted into the database
>
> If so, then we want to navigation to the EditStationView passing in the new station for editing.

There are a couple of ways to do this, but the most strait forward is to do the foillowng

### StationListView

1. In the EditStationVIew, create a State variable called newStation that is an optional Station object.  By being optional, it means that it will be nil initially.

```swift
@State private var newStation: Station?
```

2. In the .sheet modifier, add an onDismiss argument that will be a closure where you can perform an action

```swift
.sheet(isPresented: $createNewStation, onDismiss: {
   
}) {
    NewStationView()
}
```

### NewStationView

1. NewStationView will have to receive that newStation as a Binding so at the top of that view create that binding as an optional Station.  Remember it will be nil when initially passed in to the view (we will do that shortly)

```swift
@Binding var newStation: Station?
```

2. Still in NewStationView, in the action where you create the new station, before you dismiss the view, assign the newStation created to the version that is passed in

   ```swift
   Button("Create") {
       let newStation = Station(callSign: callSign, name: name)
       context.insert(newStation)
       try? context.save()
       self.newStation = newStation // This is what gets added
       dismiss()
   
   }
   ```

   > Note:  in the action above, self.newStation means the variable that is at the top of the view.  self means the entire view while newStation is the local variable within the button action.  We need to use self.newStation = newStation to distinguish between the two of them because they both have the same name

3. The preview will break because it is expecting a binding for the optional NewStation.  We can fix this by simulating the passing in of a nil newStation,.  This is done by using the @Previewable macro to create the same state property that we created in the StationListView
   1. This is then passed in to the preview so it does not break 

```swift
#Preview(traits: .mockData) {
    @Previewable @State var newStation: Station?
    NewStationView(newStation: $newStation)
}
```

### Back in StationListView

1. Back in StationListView, there will be a warning because the call to NewStationView is missing an argument which is s binding to an optional Station object.  So in the .sheet presentation closure, add the binding to newStation as the argument

```swift
.sheet(isPresented: $createNewStation, onDismiss: {
   
}) {
    NewStationView($newStation)
}
```

In this same .sheet presentation, in the onDismiss action closure, we only want to do something if the user did not tap the cancel button when the NewStationView was dismissed.  We will know that this is the case when the newStation variable is no  longer nil.  (If the user tapped on the create button we are assign that new station to the newStation Binding in that view which means that the newStation variable in the StationListView is updated too.)

2. You can use an if let to check to see if newStation is no longer optional.  This serves two purposes, not only does it check, but it also unwraps the options and assigns it to that let variable

```swift
.sheet(isPresented: $createNewStation, onDismiss: {
   if let newStation {
     
   }
}) {
    NewStationView($newStation)
}
```

> Note: The following two code blocks are equivalent but the first version is cleaner as we do not have to create a second variable name
> ```swift
> if let newStation {
>   
> }
> and
> if newStation != nil {
>   let newStation2 = newStation!
> }
> ```

#### Setting up the navigation

We now need to navigation to the EditStationView but we cannot use a navigationLine,  We need to use programmatic navigation.

I cover that in this video

 NavigationStack 2: Programmable NavigationPath and DeepLinks
https://youtu.be/pwP3_OX2G9A

1. In **StationListView** add a new State variable called path and initializer as a NavigationPath object

```swift
@State private var path = NavigationPath()
```

2. This path argument needs to be added to the NavigationStack in StationListView as a binding when it is created

```swift
NavigationStack(path: $path) {
  
}
```

> Note, the way a navigation stack works is that if there is a path define at the root level of the NavigationStack, time you use NavigationLink as we do in the sub new (StationList) what ever object is being passed in gets added to the path argument in what is called the stack.  When it returns from the destination, it is removed from the path.
>
> What we can do here is programatically add the station to the path to force a navigation to the EditStationView

3. Back agin in the sheet's onDismiss action closure,  you can append the newStatin to the path to force the navigation to occur and then immediately set newStation back to nil to prevent the next time that someone tries to add a new station and taps cancel to force a repeat navigation
   The entire sheet method now should look like this.

```swift
.sheet(isPresented: $createNewStation, onDismiss: {
    if let newStation { 
        path.append(newStation)
        self.newStation = nil
    }
}) {
    NewStationView(newStation: $newStation)
}
```

This is all fine, except we have yet to let the view know where to go when the path has a new station added to the stack,

4. This is done by attaching a navigationDestination modifier to the sheet that is triggered any time a State Station object in the view is changed.  This give us a station object that we can use in the closure

```swift
.navigationDestination(for: Station.self, destination: { station in
    
})
```

5. Since we only have a single observed Station object in our view which is that newStation State variable, then the station that we get in our closure must be that one so we can use it to pass in to, and present the NewStationView
   The completed navigationDestination method should look like this 

```swift
.navigationDestination(for: Station.self, destination: { station in
    EditStationView(station: station)
})
```
