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
Let's clear some basics of flutter before starting out. So Flutter is native how? Well unlike any other cross-platform technologies like Cordova or sencha which uses a web view to display contents, flutter on other hand uses Skia engine to lay things out without going to native platforms and that's why it is fast and fast enough to render 60fps-120fps (although this is different topic yes benchmarks are available). 

Coming back to the main topic, due to rendering natively using Skia engine flutter creates so file(for android and arm binaries for ios) which ships in platform apk/ipa files along with the flutter engine itself, you may ask why flutter engine? well, the flutter engine is the SDK or kind of like platform which helps widgets to draw things internally by converting it to skia related code, and that is why basic flutter app is about ~8-9mb as flutter engine itself is present in that so file. I am not having an idea about decompiling ipa file but if you look closely android apk file and [decompile apk] from the web or from [ShowJava app] or going with jar files any of those will decompile and give the source present in that apk. after that look at lib folder there, you will be having libflutter.so.

> Off topic but if you have .so files or arm binaries and want to reduce apk/ipa size, you can split apks just like android and publish them using same old tactics.

Again coming back to the topic, so what comes in your mind if you have .so file or arm binaries? Well, you need to load it and then use it. ```System.loadLibrary("flutter")``` to use so file or so lib we need to create a static block in Android and then write this kind of line to load that library and then use it once it is ready.

Do you remember why you need to write your app class extending ```FlutterApplication``` class? or Why you need to write this line ```FlutterMain.startInitialization(this);``` to procced with flutter? It's because internally after settings (i.e. stack trace options and profiling options and all) it uses that ```System.loadLibrary``` line to load flutter so file.

>Note: If you don't extend the application class to FlutterApplication or FlutterMain.startInitialization it might give you some error referring to Flutter library not initialized or loaded yet. (do not know the exact error but will give some error like this)

> Also there's one thing to mention, the loading library is on the main thread. Yes, libraries will only be initialized on the main thread. otherwise, it will throw error ```IllegalStateException("startInitialization must be called on the main thread")``` read FlutterLoader class for more details.

Now that the library has been loaded and activity needs to start the first thing to notice is activity extends ```FlutterActivity``` and having this line in onCreate method ```GeneratedPluginRegistrant.registerWith(this)``` so what it basically used for?
Android/Ios basically works with the view, without view there's no app (technically you can say background services but still you need to have notification running like song notification service or something). so this view is called FlutterView in flutter's context. The FlutterView will get created and at that point in the time out the main file will get executed and so is ```runApp``` method in that main function.
> Note that as OpenGL works with ```SurfaceView``` flutter also working using this ```SurfaceView``` check [FlutterView file] for more details.

And the method ```GeneratedPluginRegistrant.registerWith(this)``` is used for plugins registry. So basically it's just like linking things take an example when you request for permission fir ```MethodChannel``` will pass a message to the plugin that this is the request please take care of it.
>Note: [MethodChannel] is a class that is used to communicate between flutter and native code. Like in React-Native we use bridges this is kind of that.

> [read full detailed article of ```MethodChannel```]

[Watch how to create a plugin in flutter] for details on GeneratedPluginRegistrant.
[A little documentation of how plugins communicates with native code] read it before going ahead because it will give bost to next things.

Now that we know how to create plugin and how plugins communicate from dart to native and vice versa. Let's go ahead.

Create a new plugin using Android-Studio or command line.

>Note if you create a project via command line you might need to add below lines in your pubspec.yaml

<script src="https://github.com/parthdave93/FlutterHeadlessView/blob/master/pubspec.yaml"></script>

This is crusial part as this will enable flutter that this Plugin is there with package and you need to call this class to perform operations for that plugin.

Next is to create class in dart with static functions, so that any one can call and initialize it.

Reference link:
[Headless View sample repository]


[Background Fetch Geofencing medium article]:<https://medium.com/flutter/executing-dart-in-the-background-with-flutter-plugins-and-geofencing-2b3e40a1a124>
[decompile apk]:<http://www.javadecompilers.com/apk>
[ShowJava app]:<https://play.google.com/store/apps/details?id=com.njlabs.showjava&hl=en_IN>
[read full detailed article of ```MethodChannel```]:<./flutter_method_channel.md>
[Watch how to create a plugin in flutter]:<https://www.youtube.com/watch?v=TZRpCGQsBCw>
[A little documentation of how plugins communicates with native code]:<./flutter_generated_plugin_registrant.md>

[Headless View sample repository]:<https://github.com/parthdave93/FlutterHeadlessView>