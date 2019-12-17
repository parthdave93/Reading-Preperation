# Important Note
Most of these questions have been asked in an interview or fetched from a very helpful repository:
[refer this link](https://github.com/whatsupcoders/Flutter-Interview-Questions)
Please please do not forget to star it as he/they is/are working hard to get updated questions from the flutter community and other flutter devs as well.

----

#### What is the difference between Stateless and Stateful Widget in Flutter?
A stateful widget can be used when we need to change UI or perform an event. Or another way to put it in the widget that can rebuild itself. A stateless widget can not rebuild itself. [refer this link](https://stackoverflow.com/a/47502202)
![alt text](https://i.stack.imgur.com/g6YEk.png)

#### Tricky Question from the above statement, referring to you said stateless widget will not have any states and we need to use stateful widget to perform some operations. Will you be able to navigate or perform click event on a `Button` which resides in a Stateless widget?
Both answer is yes, as the widget can not build it self but can handle actions like button clicks and even navigation to another screen.
```Dart
class DummyHome extends StatelessWidget{
 @override
 Widget build(BuildContext context) {
 return Center(
 child: RaisedButton(child: Text("button"), onPressed: (){
 print("event");
 Navigator.push(
 context,
 MaterialPageRoute(builder: (context) => DummyHome()),
 );
 },),
 );
 }
}
```

#### Why Are StatefulWidget and State Separate Classes?
State objects are long-lived, like for an example on configuration change the StatefulWidget

--- 

#### Explain Stateful Widget Lifecycle?
The lifecycle has the following simplified steps:
1. createState()
2. mounted == true
3. initState()
4. didChangeDependencies()
5. build()
6. didUpdateWidget()
7. setState()
8. deactivate()
9. dispose()
10. mounted == false

[refer this link](https://flutterbyexample.com/stateful-widget-lifecycle/)

--- 

#### Hot reload vs Hot Restart
What is Hot Reload in Flutter:

1. Hot reload, reloads the updated file while Hot restart sends a fresh app to the device discarding the changes.
2. Hot reload compiles only updated file and sends to the device and so device preserves the state or screen with the updated values like a counter or so... While Hot restart destroys state value and sets them to their default values and so values like the counter in the sample app turns to 0. 
3. In Hot reload just build method will get called after updating Dart virtual machine while in Hot restart screen will initialize itself again and so the state will be lost and the app widget tree needs to rebuild itself.
4. Speed vise Hot reload takes much less time than Hot restart.

--- 

#### What is an Inherited Widget and list some examples?
We are using Inherited widgets too many times without knowing like:
```Theme.of(context).appBarTheme``` or ```MediaQuery.of(context).size``` or any of the components that your extending. This basics of Inherited widget has been used by provider pattern.
The inherited widget helps us to reduce boilerplate code and pass data to any of the children available.
take an example here we have
Parent( contains ABCList,ABCNumber,ABCCategory) -> Child1 -> Child2 -> Child3
Now if we have this type of a hierarchy and when we need to have listed in Child3 what you might do as an intermediate developer? develop some static function to pass-down the data (which reduces the testability) or pass down from Child1, Child2 to Child3 in the constructor.
What if you can archive all this without doing much of this hassle?
Create a custom class extending InheritedWidget
```Dart
class MyClass extends InheritedWidget{
 /// init all the variables

 /// also do not just write true here this function updates or rebuilds the widget when this condition is true so even some basic value changed then child widgets will get updated so use it carefully.
 @override
 bool updateShouldNotify() => true
}

/// Get data or variables from any child who's parent is MyCalss
class Child3 extends StatelessWidget{
 ///build method
 final fetchNearestParent = context.inheritedWidgetOfExactType(MyClass);
 /// use it in text.color = fetchNearestParent.List.length() something like this
}
```
[refer this link](https://www.youtube.com/watch?v=ml5uefGgkaA)

#### How can we control the flow of data in InheritedWidget? Suppose you have some child that needs to be updated when "this/some" property updates but updateShouldNotify method resides in parent so all widget will get rebuild how can we control it?
Use ```InheritedModel```
Will add Gist Link here in future.

--- 
#### Show, Hide and as keywrods in Dart
as and show are two different concepts.

With as you are giving the imported library a name. It's usually done to prevent a library from polluting your namespace if it has a lot of global functions. If you use as you can access all functions and classes of said library by accessing them the way you did in your example: GoogleMap.LatLng.

With show (and hide) you can pick specific classes you want to be visible in your application. For your example it would be:

```import 'package:google_maps/google_maps.dart' show LatLng;```
With this you would be able to access LatLng but nothing else from that library. The opposite of this is:

```import 'package:google_maps/google_maps.dart' hide LatLng;```
With this you would be able to access everything from that library except for LatLng.

If you want to use multiple classes with the same name you'd need to use as. You also can combine both approaches:

import 'package:google_maps/google_maps.dart' as GoogleMap show LatLng;
[refer this link](https://stackoverflow.com/a/19723473/4029614)

