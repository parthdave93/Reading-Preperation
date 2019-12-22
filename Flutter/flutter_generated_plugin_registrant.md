#### GeneratedPluginRegistrant
When we create a sample project like that counter sample, We have one Java and one Kotlin folder(i.e. if you have selected kotlin language). In Java folder there will be one file called ```GeneratedPluginRegistrant``` Which will having this type of code.

```Dart
public final class GeneratedPluginRegistrant {
  public static void registerWith(@NonNull FlutterEngine flutterEngine) {
  }
}
```

Which we are using in ```MainActivity``` or something relevant launch activity. like below
```
class MainActivity: FlutterActivity() {
    override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        GeneratedPluginRegistrant.registerWith(flutterEngine);
    }
}
```

and so we are calling that GeneratedPluginRegistrant to register the plugins. long time ago this method was not automatic or there was no auto generated plugin registrant and so developer needs to do manual call to all the plugin to register.
kepping that aside.

Currently we dont have any plugin that's why it is empty. After adding one We might have something like this:

```Dart
public final class GeneratedPluginRegistrant {
  public static void registerWith(@NonNull FlutterEngine flutterEngine) {
    ShimPluginRegistry shimPluginRegistry = new ShimPluginRegistry(flutterEngine);
      com.baseflow.permissionhandler.PermissionHandlerPlugin.registerWith(shimPluginRegistry.registrarFor("com.baseflow.permissionhandler.PermissionHandlerPlugin"));
  }
}

```
Here I have added backgroundfetch plugin in my pubspec.yaml file.

There are specifically 2 classes for library
1. class which implements PluginRegistry - for register plugin or mapping
2. class extending MethodChannel - for communicating with the dart code.

So here in ```ShimPluginRegistry shimPluginRegistry = new ShimPluginRegistry(flutterEngine);``` , ShimPluginRegistry is a class which implements ```PluginRegistry``` this class takes reference to the FlutterEngine which will give the dartExecutor, activity result callback and other variables which will be used to get the data and then MethodChannel to post data back to the dart code.

Basically every Plugin will get registered for their package name and there will be one HashMap which maps with the plugins that are registered.

Take a look at [PermissionHandlerPlugin] to which GeneratedPluginRegistrant calls one method registerWith with the Registrar which creates MethodChannel and sets listeners like permissionHandler and activityResult listener.


```
public static void registerWith(Registrar registrar) {
    final MethodChannel channel = new MethodChannel(registrar.messenger(), "flutter.baseflow.com/permissions/methods");
    final PermissionHandlerPlugin permissionHandlerPlugin = new PermissionHandlerPlugin(registrar);
    channel.setMethodCallHandler(permissionHandlerPlugin);

    registrar.addRequestPermissionsResultListener(new PluginRegistry.RequestPermissionsResultListener() {
      @Override
      public boolean onRequestPermissionsResult(int id, String[] permissions, int[] grantResults) {
        if (id == PERMISSION_CODE) {
          permissionHandlerPlugin.handlePermissionsRequest(permissions, grantResults);
          return true;
        } else {
          return false;
        }
      }
    });

    registrar.addActivityResultListener(new ActivityResultListener() {
      @Override
      public boolean onActivityResult(int requestCode, int responseCode, Intent intent) {
        if (requestCode == PERMISSION_CODE_IGNORE_BATTERY_OPTIMIZATIONS) {
          permissionHandlerPlugin.handleIgnoreBatteryOptimizationsRequest(responseCode == Activity.RESULT_OK);
          return true;
        }

        return false;
      }
    });
  }
```


Let's take steps 
When we request for permission [Permission Handler dart] code is called.
```
class PermissionHandler {
  factory PermissionHandler() {
    if (_instance == null) {
      const MethodChannel methodChannel =
          MethodChannel('flutter.baseflow.com/permissions/methods');

      _instance = PermissionHandler.private(methodChannel);
    }
    return _instance;
  }
  
  ...

/// Request the user for access to the supplied list of permissiongroups.
  ///
  /// Returns a [Map] containing the status per requested permissiongroup.
  Future<Map<PermissionGroup, PermissionStatus>> requestPermissions(
      List<PermissionGroup> permissions) async {
    final List<int> data = Codec.encodePermissionGroups(permissions);
    final Map<dynamic, dynamic> status =
        await _methodChannel.invokeMethod('requestPermissions', data);

    return Codec.decodePermissionRequestResult(Map<int, int>.from(status));
  }
```
as you can see it calls ```_methodChannel.invokeMethod('requestPermissions', data);``` so all communication done via method channels. and eventually native Android/Ios code will going to be executed and result will be passed via result.

```
@Override
  public void onMethodCall(MethodCall call, Result result) {
    switch (call.method) {
      case "checkPermissionStatus": {
        @PermissionGroup final int permission = (int) call.arguments;
        @PermissionStatus final int permissionStatus = checkPermissionStatus(permission);

        result.success(permissionStatus);
        break;
      }
```

>Note every plugin I mean every plugin needs to have it's own MethodCallHandler which will be called when MethodChannel receives the event. for method channel check Method channel doc in this repo.

[Permission Handler dart]:<https://github.com/Baseflow/flutter-permission-handler/blob/develop/lib/src/permission_handler.dart>
[PermissionHandlerPlugin]:<https://github.com/Baseflow/flutter-permission-handler/blob/develop/android/src/main/java/com/baseflow/permissionhandler/PermissionHandlerPlugin.java>