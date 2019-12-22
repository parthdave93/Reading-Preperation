#### Important Note
I'm not a certified flutter developer but I will share everything that I have learned using reading and exploring the base classes of the frameworks. Please feel free to create an issue if any detail is wrong or misleading, will be happy if you do so.

---

# HeadlessView
#### What we want to archive through this topic?
First let's take an example, Suppose you have some app which having alarm manager and wakes up the app 
The possibilities:
- fetch data from the web and stores it locally 
- update data on a server
- or we want to connect the device with some Bluetooth or any other device and do some computing like a wearable watch and fetch data of it(i.e. running, burned calories and all)

So we will be having a flutter database or some flutter code that needs to run with native code to sync things up. Like connecting to a smartwatch, get data from it and store it locally(obviously you will not create two diff database for android/ios right?) At that very moment you need to create ```Headless View```.

#### Background Fetch using Native Android/Ios Code
There is officially no document to do this stuff.
Alright alright, there is an article [Background Fetch Geofencing medium article] but it's too much complex and left the main path to explain how to do this stuff.

##### Basics of flutter
Let's clear some basics of flutter before starting. So Flutter is native how? Well unlike any other cross-platform technologies like Cordova or sencha which uses a web view to display contents, flutter on other hand uses Skia engine to lay things out without going to native platforms and that's why it is fast and fast enough to render 60fps-120fps (although this is different topic yes benchmarks are available). 

Coming back to the main topic, due to rendering natively using Skia engine flutter creates so file(for android and arm binaries for ios) which ships in platform apk/IPA files along with the flutter engine itself, you may ask why flutter engine? well, the flutter engine is the SDK or kind of like platform which helps widgets to draw things internally by converting it to skia related code, and that is why basic flutter app is about ~8-9mb as flutter engine itself is present in that so file. I am not having an idea about decompiling the IPA file but if you look closely android apk file and [decompile apk] from the web or from [ShowJava app] or going with jar files any of those will decompile and give the source present in that apk. after that look at lib folder there, you will be having libflutter.so.

> Off topic but if you have .so files or arm binaries and want to reduce apk/IPA size, you can split apks just like android and publish them using same old tactics.

Again coming back to the topic, so what comes in your mind if you have .so file or arm binaries? Well, you need to load it and then use it. ```System.loadLibrary("flutter")``` to use so file or so lib we need to create a static block in Android and then write this kind of line to load that library and then use it once it is ready.

Do you remember why you need to write your app class extending ```FlutterApplication``` class? or Why you need to write this line ```FlutterMain.startInitialization(this);``` to procced with flutter? It's because internally after settings (i.e. stack trace options and profiling options and all) it uses that ```System.loadLibrary``` line to load flutter so file.

>Note: If you don't extend the application class to FlutterApplication or FlutterMain.startInitialization it might give you some error referring to Flutter library not initialized or loaded yet. (do not know the exact error but will give some error like this)

> Also there's one thing to mention, the loading library is on the main thread. Yes, libraries will only be initialized on the main thread. otherwise, it will throw error ```IllegalStateException("startInitialization must be called on the main thread")``` read FlutterLoader class for more details.

Now that the library has been loaded and activity needs to start the first thing to notice is activity extends ```FlutterActivity``` and having this line in onCreate method ```GeneratedPluginRegistrant.registerWith(this)``` so what it used for?
Android/Ios works with the view, without view there's no app (technically you can say background services but still you need to have notification running like song notification service or something). so this view is called FlutterView in flutter's context. The FlutterView will get created and at that point in the time out the main file will get executed and so is ```runApp``` method in that main function.
> Note that as OpenGL works with ```SurfaceView``` flutter also working using this ```SurfaceView``` check [FlutterView file] for more details.

And the method ```GeneratedPluginRegistrant.registerWith(this)``` is used for plugins registry. So basically it's just like linking things take an example when you request for permission fir ```MethodChannel``` will pass a message to the plugin that this is the request please take care of it.
>Note: [MethodChannel] is a class that is used to communicate between flutter and native code. Like in React-Native we use bridges this is kind of that.

> [read full detailed article of ```MethodChannel```]

[Watch how to create a plugin in flutter] for details on GeneratedPluginRegistrant.
[A little documentation of how plugins communicate with native code] read it before going ahead because it will give a boost to the next things.

Now that we know how to create a plugin and how plugins communicate from dart to native and vice versa. Let's go ahead.

Create a new plugin using Android-Studio or command line.

>Note if you create a project via command line you might need to add below lines in your pubspec.yaml

[flie link](https://github.com/parthdave93/FlutterHeadlessView/blob/master/pubspec.yaml)
```YAML
flutter:
 plugin:
 androidPackage: com.yourPackage
 pluginClass: YourPluginName
```
This is a crucial part as this will enable flutter that this Plugin is there with the package and you need to call this class to perform operations for that plugin.

Next is to create a class in dart with static functions, so that anyone can call and initialize it.

[flie link](https://github.com/parthdave93/FlutterHeadlessView/blob/master/lib/flutter_background_plugin.dart)
```Dart

void backgroundIsolateMain(){
 print("Flutter side \n\n\n\n backgroundIsolateMain");
}

class FlutterBackgroundPlugin {
 static const MethodChannel _channel =
 const MethodChannel('flutter_background_plugin');

 static Future<String> get platformVersion async {
 final String version = await _channel.invokeMethod('getPlatformVersion');
 return version;
 }

 static initIsolates() async{
 const methodChannel = MethodChannel("platform_specific");
 var callback = PluginUtilities.getCallbackHandle(backgroundIsolateMain);
 WidgetsFlutterBinding.ensureInitialized();
 await methodChannel.invokeMethod("initBackgroundProcess",callback.toRawHandle());
 }
}
```

Here ```PluginUtilities.getCallbackHandle(backgroundIsolateMain);``` PluginUtilies provides utilities for dealing with Isolates. getCallbackHandle checks if the plugin already been added to the Map or not, if not then add callback reference to native end (i.e. Internal Native NDK).

afterward, we added one line to ```WidgetsFlutterBinding.ensureInitialized();``` this helps to bind framework to the flutter engine.

and then we invoked method channel to initBackgroundProcess. which calls HeadlessPlugin from the native end. and onMethodCall going to be invoked ``` override fun onMethodCall(call: MethodCall, result: MethodChannel.Result)``` from which we are going to executeBackgroundIsolate which saves the callback location or should I say reference of a dart.


[flie link](https://github.com/parthdave93/FlutterHeadlessView/blob/master/android/src/main/kotlin/com/parthdave93/flutter_background_plugin/HealdessPlugin.kt)
```
class HealdessPlugin(val context: Context) : MethodChannel.MethodCallHandler {

 val SHARED_PREF = "com.example.background_process"
 
 companion object {
 @JvmStatic
 fun registerWith(registrar: PluginRegistry.Registrar) {
 val channel = MethodChannel(registrar.messenger(), "platform_specific")
 val plugin = HealdessPlugin(registrar.context())
 channel.setMethodCallHandler(plugin)
 }
 }

 override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
 Log.e("method call", "method called from flutter to native: ${call.method}")
 if (call.method == "initBackgroundProcess") {
 val callbackHandle = call.arguments as Long
 executeBackgroundIsolate(context, callbackHandle)
 }else{
 result.success(true)
 }
 }

 private fun executeBackgroundIsolate(context: Context, callbackHandle: Long) {
 val preferences = context.getSharedPreferences(SHARED_PREF, IntentService.MODE_PRIVATE)
 preferences.edit().putLong("callback", callbackHandle).apply()
 }
}
```

[flie link](https://github.com/parthdave93/FlutterHeadlessView/blob/master/android/src/main/kotlin/com/parthdave93/flutter_background_plugin/UploadWorker.kt)
```
class UploadWorker(appContext: Context, workerParams: WorkerParameters)
 : Worker(appContext, workerParams), MethodChannel.MethodCallHandler {
 val SHARED_PREF = "com.example.background_process"
 val obj = Object()

 override fun doWork(): Result {
 // Do the work here--in this case, upload the images.

 Log.e("method call", "UploadWorker->doWork")
 synchronized(obj){
 Handler(Looper.getMainLooper()).post {
 startBackgroundIsolate()
 }
 }
 Log.e("method call", "UploadWorker->return success")
 // Indicate whether the task finished successfully with the Result
 return Result.success()
 }

 private fun startBackgroundIsolate() {
 Log.e("method call", "UploadWorker->startBackgroundIsolate")

 FlutterMain.ensureInitializationComplete(applicationContext, null)

 val preferences = applicationContext.getSharedPreferences(SHARED_PREF, IntentService.MODE_PRIVATE)
 val callbackHandle = preferences.getLong("callback", 0)
 if(callbackHandle == 0L)
 return
 val callbackObj = FlutterCallbackInformation.lookupCallbackInformation(callbackHandle)
 val backgroundProcessView = FlutterNativeView(applicationContext, true)
 val path = FlutterMain.findAppBundlePath()
 val args = FlutterRunArguments()
 args.bundlePath = path
 args.entrypoint = callbackObj.callbackName
 args.libraryPath = callbackObj.callbackLibraryPath

 backgroundProcessView.runFromBundle(args)
 val backgroundChannel = MethodChannel(backgroundProcessView, "backgroundCallChannel")
 backgroundChannel?.setMethodCallHandler(this)
 App.pluginregistrantCallbacks.registerWith(backgroundProcessView.pluginRegistry)
 }

 override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
 Log.e("method call", "method called from flutter to native: ${call.method}")
 obj.notify()
 result.success(true)
 }
}
```

Now in UploadWorker, we have started Isolate or should I say Headless View when UploadWorker gets invoked which is 2minutes after an application was launched. (remember we also have updated app file and it's one-time request so kill the app and wait for 2minutes.)

To test things working file we have changed Application class to this

```
class App : FlutterApplication(), PluginRegistry.PluginRegistrantCallback {
 override fun registerWith(registry: PluginRegistry?) {
 GeneratedPluginRegistrant.registerWith(registry)
 }

 companion object{
 const val SHARED_PREF = "shared_pref"
 lateinit var appInstace: App
 lateinit var pluginregistrantCallbacks: PluginRegistry.PluginRegistrantCallback
 }

 @RequiresApi(Build.VERSION_CODES.GINGERBREAD)
 override fun onCreate() {
 super.onCreate()
 appInstace = this
 pluginregistrantCallbacks = this
 val uploadWorkRequest = OneTimeWorkRequestBuilder<UploadWorker>()
 .setInitialDelay(2, TimeUnit.MINUTES)
 .build()
 WorkManager.getInstance(this@App).enqueue(uploadWorkRequest)
 }
}
```

Check doWork method, so when 2minute is finished UploadWorker will get invoked by the system and we are invoking ```startBackgroundIsolate()``` so that we can start executing isolate.

>Note we have used ```synchronized(obj)``` to wait until we notify the success and execute the next line of startBackgroundIsolate. 

``` FlutterMain.ensureInitializationComplete(applicationContext, null)``` line is important as sometimes Flutter lib may not be initialized so we are telling the framework to initialize before moving forward. Then we need to have a reference, by reference means flutter invoke reference which we have stored in prefs. 

```val callbackObj = FlutterCallbackInformation.lookupCallbackInformation(callbackHandle)``` will check if callback is available or not on that reference. Now create FlutterNativeView, the difference between FlutterView and FlutterNativeView is FlutterView used FlutterNativeView and it's not related to drawing sequence. FlutterNativeView is purely for dispatch events like native to dart call and allocations and deallocations of JNI. while FlutterView extends to SurfaceView so it needs to be connected with UI. 

There are two parameters in FlutterNativeView one is context and the second one is a boolean flag to giving info to NativeView that this is background initialization and not connected with UI by any chance.

after that, we need to have AppBundlePath, FlutterRunArguments and entry point which can be found using that reference we talked about.

>Note these arguments are important as without this you will not be able to invoke dart function.

then create MethodChannel and call with the binaryMessenger which in our case is NativeView with the method name. and a callback to handle the result.

```
val callbackObj = FlutterCallbackInformation.lookupCallbackInformation(callbackHandle)
val backgroundProcessView = FlutterNativeView(applicationContext, true)
val path = FlutterMain.findAppBundlePath()
val args = FlutterRunArguments()
args.bundlePath = path
args.entrypoint = callbackObj.callbackName
args.libraryPath = callbackObj.callbackLibraryPath

backgroundProcessView.runFromBundle(args)
val backgroundChannel = MethodChannel(backgroundProcessView, "backgroundCallChannel")
backgroundChannel?.setMethodCallHandler(this)
App.pluginregistrantCallbacks.registerWith(backgroundProcessView.pluginRegistry)
```

I know it's a long article but this will be going to help you if you stuck in BackgroundFetch as I was when developing the SmartWatch app. I need to invoke so file and connect with the device to fetch data which needs to have background process running with flutter code to update data on the database.

Hope this helps, Let me know the comments if I've misled or wrong understanding create an issue on this repository will update the article. Also, help the reference links as I was reading many articles and trying to understand the works to which I have been grown by these guys and Flutter Community.

Reference link:
[Headless View sample repository]


[Background Fetch Geofencing medium article]:<https://medium.com/flutter/executing-dart-in-the-background-with-flutter-plugins-and-geofencing-2b3e40a1a124>
[decompile apk]:<http://www.javadecompilers.com/apk>
[ShowJava app]:<https://play.google.com/store/apps/details?id=com.njlabs.showjava&hl=en_IN>
[read full detailed article of ```MethodChannel```]:<./flutter_method_channel.md>
[Watch how to create a plugin in flutter]:<https://www.youtube.com/watch?v=TZRpCGQsBCw>
[A little documentation of how plugins communicate with native code]:<./flutter_generated_plugin_registrant.md>

[Headless View sample repository]:<https://github.com/parthdave93/FlutterHeadlessView>