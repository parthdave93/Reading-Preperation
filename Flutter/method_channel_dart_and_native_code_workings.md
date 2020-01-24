To dismantle the flutter method channel let’s go a step back to understand why is it there in the first place.
Flutter creates a .so file or binary file in which it contains skia framework and flutter frameworks which helps to render our UI.
Flutter app always has App class for android and App delegate for IOS which is used for initializing or loading .so or arm binaries for respective platforms.
For Android:
An application class must be extended by the FlutterApplication class, Flutter Application class is defined as the default app class in Android Manifest.

![manifest](https://github.com/parthdave93/Reading-Preperation/blob/master/Flutter/carbon-9.png)

If we could not be able to extend Application class to FlutterApplication class then we need to manually call FlutterMain.startInitialization() which will initialize the flutter binary files and load it to the system, and then start the flutter main.dart file main function.
![application class](https://github.com/parthdave93/Reading-Preperation/blob/master/Flutter/carbon-10.png)

In IOS I was not able to find out if we can be able to launch AppDelegate with UIResponder and UiAppDelegate and just like android call init or something. (feel free to comment if you found out how)
Here GeneratedPluginRegistrant is for plugins, it creates a map with the invoker function for a plugin, If flutter calls native then lib will send data in method channel and method channel will invoke the function (It’s a little different with background fetch plugin but almost same for others).
![ios app delegate](https://github.com/parthdave93/Reading-Preperation/blob/master/Flutter/carbon-8.png)

Why we need a native bridge?
What if you want to call .so or .arm binary file code? Like we usually do for a smart band.
Pre-compiled source code or existing logic which already been developed before.
What is a native bridge?
As we read before flutter creates a binary file due to which it’s impossible to call Java/Objective C code from JNI/binary without some channel.
JNI calls are like both parties (i.e. Java and C++ class file) have methods that they can call. If we think like this then we need to create files and methods for every possibility out there. And so we need common logic from which we do not need to declare methods in both ends but able to use it without doing so which is where the channel comes into the picture.
How to do a native bridging?
MethodChannel is a class that is used to communicate between flutter and native code. Like in React-Native we use bridges this is kind of that or IBinder for Binding service or Delegates.
Create a platform Channel
static const platform = const MethodChannel('channel_name');
A channel name is like a unique key for code on both ends.
Invoke method on platform Channel
static const platform = const MethodChannel('channel_name');
platform_channel() async{
  String response = "";
  try {
    final String result = await  platform.invokeMethod('method_name', arguments);
    response = result;
  } on PlatformException catch (e) {
    response = "Failed to Invoke: '${e.message}'.";
  }
}
here method name will be retrieved when we got a call from method channel on native end. arguments are converted to bytes internally from string and then you can get a string from the native end. I will explain await after some time.
Android platform channel method implementation
val handler = object: MethodChannel.MethodCallHandler{
  override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
    if (call.method.equals("method_name")) {
      result.success(result_to_send);
    }
  }
};

MethodChannel(flutterView, "channel_name").setMethodCallHandler(handler);
result_to_send will send data back to the flutter code and await there was like async function call so until the result not fetched it will wait for it.
IOS platform channel method implementation
let channelName = "channel_name"
    let rootViewController : FlutterViewController = window?.rootViewController as! FlutterViewController
    
    let methodChannel = FlutterMethodChannel(name: channelName, binaryMessenger: rootViewController)

    methodChannel.setMethodCallHandler {(call: FlutterMethodCall, result: FlutterResult) -> Void in
        if (call.method == "method_name") {
            result.success(result_to_send)
        }
    }
And voila! we are done with the implementation. easy peasy lemon squeezy right?
