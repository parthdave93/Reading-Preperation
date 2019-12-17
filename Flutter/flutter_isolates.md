# Dart is a single-threaded system
---
Dart is a single-threaded system. Sometimes we have hard times using this as now every language is using a multi-threaded system and dart uses old concepts but it's not over yet and Dart is still evolving. The main reason to take this point is to start the Flutter Isolates but let's first check why we need to Isolate.

>IMPORTANT
Dart executes one operation at a time, one after the other meaning that as long as one operation is executing, it cannot be interrupted by any other Dart code.

Create a default new flutter app that will create an increment counter template.
```Dart
@override
 Widget build(BuildContext context) {
  return Scaffold(
   appBar: AppBar(
    title: Text(widget.title),
   ),
   body: Center(
    child: Column(
     mainAxisAlignment: MainAxisAlignment.center,
     children: <Widget>[
      CircularProgressIndicator(),
      Text(
       'You have pushed the button this many times:',
      ),
      Text(
       '$_counter',
       style: Theme.of(context).textTheme.display1,
      ),
     ],
    ),
   ),
   floatingActionButton: FloatingActionButton(
    onPressed: _incrementCounter,
    tooltip: 'Increment',
    child: Icon(Icons.add),
   ), // This trailing comma makes auto-formatting nicer for build methods.
  );
 }
```
We have added circularProgressIndicator just to check when the UI stucks or skips frame. Let's update incrementCounter function.
```Dart
void _incrementCounter() {
  setState(() {
   var string = Constants.str;
   for(int index = 0;index <10;index++) {
    for (int i = 0; i < string.length; i++) {
     string += string[i];
    }
   }

   _counter++;
  });
 }
```
Click on the FAB button, the circular progress bar stuck for a while and loads again. This is due to heavy lifting on the main thread(or I can say the Main isolate, Yah's main thread is here in flutter is Main Isolate). Well, you will probably say "Ahhh!! It's easy, just put this code in an async function and call it from there" Well that is correct but it's not going to work. Why?
because Dart works on event-loop mechanism what it means is it creates events for the main thread or any isolate and runs the event priority basis.

#### Event loop

The event loop is kind of like an infinite queue or loop, which runs forever.
Example:
create an async function and call it. at the point of time what Event loop contains is this sequence of information.
1. return an output of the method i.e. Future 
2. run async function synchronously up to the first await keyword.
3. wait for that async function to finish and execute the remaining code.

> An async method is NOT executed in parallel but following the regular sequence of events, handled by the Event Loop, too.

so if we have 
```Dart
main(){
  method1();
  method2();
}
method1() async{
  print("1");
}
method2() async{
  Future((){
    print("Future 2"); 
  });
  print("2");
}
```
output:
```
1
2
Future 2
```
So in the main isolate what event loop contains is like this
1. call method1
2. call method 2
3. Add Future event from method 2
4. execute a print statement
5. execute future event from method 2

Why Future from method 2 not run before print 2? because every entry point goes inside the event loop and as the futures will not be needed at that point of time the next sequence will get executed before futures. If we tweak code a little bit it will show a different result.

```Dart
method2() async{
  await Future((){
    print("Future 2"); 
  });
  print("2");
}
```
Now it will print 
```
1
Future 2
2
```
As event loop come to know that it needs to await for the result before executing next statement.

> After executing the main method every change has to go through the event loop to update the data. and so you might feel lag in UI when building the Flutter app due to this. Like doing IO operation or bitmap manipulation is kind of CPU or Disk process and it takes a lot of time inside the event loop so the next frame will not get updated and UI will be stuck for a while. Check ```--profile``` flag for more details to check the profile tooling in flutter(Will cover how to check this later in the post).


Reference Links:
1. [Isolate and event loop intro video from Flutter Devs]

# Isolates
---
An Isolate is a Thread. Isolates do not share memory as we have covered the main isolate have their event loop, it's the same for other isolates also and so to send and receive processed data you need to use Ports to communicate back-and-forth.

>“Isolates” in Flutter do not share memory. Interaction between different “Isolates” is made via “messages” in terms of communication.

You can create isolate 2 ways:
1. Use compute function and it will handle all communication process for data sending and receiving.
2. Write all logic to send and receive data back-and-forth your self.

> Isolates are kind of TCP Socket. We need to create a handshake mechanism for this. There is also another reason to do this is, as we define earlier Isolate does not share memory and you need to use ports and messages to pass data back-and-forth. That is why we need to use some kind of handshake.

There are some rules for creating Isolate
1. Isolates need to have a non-static top-level function.
2. If you use ```compute``` function or ```Isolate.spawn``` function for now function must have one parameter.
  You may ask why? check the method signature.
  ```Dart
    external static Future<Isolate> spawn<T>(
      void entryPoint(T message), T message,
      {bool paused: false,
      bool errorsAreFatal,
      SendPort onExit,
      SendPort onError,
      @Since("2.3") String debugName});
  ```
  Check ```entryPoint(T message)``` method call, It takes T message and so you are required to have a single parameter function.
3. Isolates take up around ~2MB per Isolate and it's lightweight so use it carefully. [Isolate memory size and allowcation] - view for more details
4. Every message pass from Isolates requires message size * 2 memory as it will be two copies on system since Isolate do not share memory, So if you have 1 GB of file and want to pass from one Isolate to second it will take 2Gb of memory on device (on device I mean RAM - check reference links for this details).
5. Isolates and Ports can be alive even though you have completed your work or all statements are executed from the method. So it's up to us to kill isolates and close the ports.

Code(No Flutter code for this pure Dart code):
```Dart
import 'dart:isolate';

void main() async{
 startIsolate();
}
```
imported Isolate package and called startIsolate function.
We will start with creating ```ReceivePort``` which is used to convey a message back and forth. The thing to remember ```ReceivePort Stream``` can only be used one time, so if we send data one time and taking data from ```receivePort.first``` now when we send data again and try to get data from ```receivePort.first``` it will not work and will throw an error with the statement:
```Unhandled Exception: Bad state: Stream has already been listened to.```
The most common form of Stream can listen only once at a time. If you try to add multiple listeners it will throw an exception. so we will use ```receivePort.first``` to get ```SendPort``` back and forth and if we want to use just one ```ReceivePort``` for better optimization we need to use listen to method from that class.
Steps to the handshake (Will give dummy examples to have a clear idea of what classes are used for)
1. Create Receive Port object (example: Receive Port is kind of like exposed server)
2. Spawn Isolate with the send port (example: Send port is kind of like a port like a socket 8080 port where we launch our socket and others can access it by calling website...:8080 and server will give the response to that request on that port)
3. In Isolate we need to use a handshake, What I mean by that is We have 2 threads and we need to communicate messages so we need to have some stream where we listen for both sides of events and so we will pass sendPorts first to establish where we need to send the message.
4. So after creating Isolate by spawn, we will wait for Isolate to send its SendPort, to which we will ping for msg.
5. At the side of Isolate, we will again create ReceiverPort and send a message to the SendPort we have passed in method argument, and the message would be the SendPort of this Isolate.
6. Remember to use ```receiverPort.first``` only one time and we will get the Send Port.
7. Now once we get all ports, It's like Socket open channel we will send messages back and forth.
```Dart
startIsolate() async{
 //create ReceivePort
 ReceivePort receivePort = ReceivePort();
 //receivePort.sendPort is our destination for receiving message
 var isolate = await Isolate.spawn(calculateFunction, receivePort.sendPort);

 SendPort sendPort = await receivePort.first;
 // we got Isolate's SendPort or should I say Message destination
}

calculateFunction(SendPort sendPort) async{
 var calculateFunctionReceivePort = ReceivePort();
 sendPort.send(calculateFunctionReceivePort.sendPort);

 await for (var msg in calculateFunctionReceivePort){
  var data = msg[0];
  SendPort replyTo = msg[1];
  replyTo.send("$data from Isolate");
  print("Received inside Isolate: $data");
 }
}
```
Once we get the sendPort of the Isolate we will send message and listen for isolate response.
```Dart
import 'dart:isolate';

void main() async{
 startIsolate();
}

startIsolate() async{
 ReceivePort receivePort = ReceivePort();
 var isolate = await Isolate.spawn(calculateFunction, receivePort.sendPort);

 SendPort sendPort = await receivePort.first;
 receivePort.close(); // closing the port , do not forget to close it.
 var msg = await sendStreamData(sendPort, "Test Data");
 print(msg);

 Future.delayed(Duration(seconds: 1),()async{
  msg = await sendStreamData(sendPort, "Test Data 2");
  print(msg);

  isolate.kill(priority: Isolate.immediate);
 });
}

sendStreamData(SendPort sendPort, String msg) async{
 ReceivePort sendStreamDataPort = ReceivePort();
 sendPort.send([msg, sendStreamDataPort.sendPort]);
 var response = await sendStreamDataPort.first;
 sendStreamDataPort.close();
 return response;
}

calculateFunction(SendPort sendPort) async{
 var calculateFunctionReceivePort = ReceivePort();
 sendPort.send(calculateFunctionReceivePort.sendPort);

 await for (var msg in calculateFunctionReceivePort){
  var data = msg[0];
  SendPort replyTo = msg[1];
  replyTo.send("$data from Isolate");
  print("Received inside Isolate: $data");
 }
}
```
output:
```
Received inside Isolate: Test Data
Test Data from Isolate
Received inside Isolate: Test Data 2
Test Data 2 from Isolate
```

### Questions:
1. Why we use ReceiverPort? or What is ReceiverPort in Isolate?
2. Why do we need to use Isolate when we have async code? 
3. How long Isolates live? and how to kill or close it?
4. Does Isolates Share memory? If no then how to pass data?

Reference Links:
1. [Isolates benchmarks profilers example and more depth]
2. [Flutter rendering pipeline]
3. [Flutter Isolates and Event-loop explaination]
4. [Isolate memory size and allowcation]
5. [Isolate and event loop intro video from Flutter Devs]

[Isolate and event loop intro video from Flutter Devs]:<https://www.youtube.com/watch?v=vl_AaCgudcY>
[Isolates benchmarks profilers example and more depth]:<https://www.youtube.com/watch?v=M8jGSkACneE>
[Flutter rendering pipeline]:<https://www.youtube.com/watch?v=UUfXWzp0-DU&feature=youtu.be>
[Flutter Isolates and Event-loop explaination]:<https://www.didierboelens.com/2019/01/futures---isolates---event-loop/>
[Isolate memory size and allowcation]:<https://www.youtube.com/watch?v=M8jGSkACneE&t=27m25s>
