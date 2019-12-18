#### MethodChannel
MethodChannel is a class that is used to communicate between flutter and native code. Like in React-Native we use bridges this is kind of that.
Take an example:
If you ever came across permission to be asked from a user as part of data fetching then you might have stumbled across the same permission lib mentioned in reference link.
Let's take a look what it internally uses (Topic will get interesting if you have basic knowledge of Android, Will try to convey something to Ios devs in future edits as I've not explored Ios native level development)
So when you call for requestPermission this [Permission dart lib file] will give a call to native code via MethodChannel.
>Internally Method Channel uses the same mechanism of Java To native code calling and native to Java code calling. [hello JNI basics of JNI] read this for reference.
```Dart
public final class MethodChannel {
 private static final String TAG = "MethodChannel#";
 private final BinaryMessenger messenger;
 private final String name;
 private final MethodCodec codec;

 public MethodChannel(BinaryMessenger messenger, String name) {
 this(messenger, name, StandardMethodCodec.INSTANCE);
 }
 ....
```
Method Channel uses BinaryMessenger this class is used to convey message, Let's take a look for BasicBinaryMessenger which is DefaultBinaryMessanger.
```
private static class DefaultBinaryMessenger implements BinaryMessenger {
 private final DartMessenger messenger;

 private DefaultBinaryMessenger(@NonNull DartMessenger messenger) {
 this.messenger = messenger;
 }

 @UiThread
 public void send(@NonNull String channel, @Nullable ByteBuffer message) {
 this.messenger.send(channel, message, (BinaryReply)null);
 }

 @UiThread
 public void send(@NonNull String channel, @Nullable ByteBuffer message, @Nullable BinaryReply callback) {
 this.messenger.send(channel, message, callback);
 }

 @UiThread
 public void setMessageHandler(@NonNull String channel, @Nullable BinaryMessageHandler handler) {
 this.messenger.setMessageHandler(channel, handler);
 }
 }
```
So all send calls to the function with channel replyId and message goes to messenger which is DartMessenger.
```
class DartMessenger implements BinaryMessenger, PlatformMessageHandler {
...
 public void send(@NonNull String channel, @Nullable ByteBuffer message, @Nullable BinaryReply callback) {
 Log.v("DartMessenger", "Sending message with callback over channel '" + channel + "'");
 int replyId = 0;
 if (callback != null) {
 replyId = this.nextReplyId++;
 this.pendingReplies.put(replyId, callback);
 }

 if (message == null) {
 this.flutterJNI.dispatchEmptyPlatformMessage(channel, replyId);
 } else {
 this.flutterJNI.dispatchPlatformMessage(channel, message, message.position(), replyId);
 }

 }
...
}
```
Then DartMessenger will send details to the flutterJni to dispatchPlatformMessage and it will eventually call the native function.
```
@UiThread
 public void dispatchPlatformMessage(@NonNull String channel, @Nullable ByteBuffer message, int position, int responseId) {
 ensureRunningOnMainThread();
 if (isAttached()) {
 nativeDispatchPlatformMessage(
 nativePlatformViewId,
 channel,
 message,
 position,
 responseId
 );
 } else {
 Log.w(TAG, "Tried to send a platform message to Flutter, but FlutterJNI was detached from native C++. Could not send. Channel: " + channel + ". Response ID: " + responseId);
 }
 }

 // Send a data-carrying platform message to Dart.
 private native void nativeDispatchPlatformMessage(
 long nativePlatformViewId,
 @NonNull String channel,
 @Nullable ByteBuffer message,
 int position,
 int responseId
 );
```
The responseId is used to identify which methodChannel method call returned data.

#### Now from Dart perspective:
```Dart
static Future<List<Permissions>> requestPermissions(List<PermissionName> permissionNameList) async {
 List<String> list = [];
 permissionNameList.forEach((p) {
 list.add(getPermissionString(p));
 });
 var status = await channel.invokeMethod("requestPermissions", {"permissions": list});
 List<Permissions> permissionStatusList = [];
 for (int i = 0; i < status.length; i++) {
 PermissionStatus permissionStatus;
 switch (status[i]) {
 case 0:
 permissionStatus = PermissionStatus.allow;
 break;
 case 1:
 permissionStatus = PermissionStatus.deny;
 break;
 case 2:
 permissionStatus = PermissionStatus.notDecided;
 break;
 case 3:
 permissionStatus = PermissionStatus.notAgain;
 break;
 default:
 permissionStatus = PermissionStatus.notDecided;
 break;
 }
 permissionStatusList.add(Permissions(permissionNameList[i], permissionStatus));
 }
 return permissionStatusList;
 }
```
first this is an asnyc function and ```await channel.invokeMethod("requestPermissions", {"permissions": list})``` this line will wait until the response received from that invokeMethod. 
```Dart
override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
}
```
this method will get called to the native end when MethodChannel gets called from dart. and as we can see we have result this is the callback for JNI so when you send some data on result.success or error the dart will receive the result on that awaiting call.
[read more on how to create method channel from the official site].

That's all, This was the basics of MethodChannel in Flutter. Hope you liked it. Feel free to create an issue if any of the detail is incorrect or incomplete or should have more explanation or you have some suggestions, will be glad to receive feedback.

##### Reference Links
- [Permission Ios code for method channel]
- [Permission Android code for method channel]
- [Permission dart lib file]

[FlutterView file]:<https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterView.java>
[MethodChannel]:<./method_channel.md>
[Permission Android code for method channel]:<https://github.com/once10301/permission/blob/master/android/src/main/java/com/ly/permission/PermissionPlugin.java>
[Permission Ios code for method channel]:<https://github.com/once10301/permission/blob/master/ios/Classes/PermissionPlugin.m>
[Permission dart lib file]:<https://github.com/once10301/permission/blob/master/lib/permission.dart>
[hello JNI basics of JNI]:<https://developer.android.com/ndk/samples/sample_hellojni>
[read more on how to create method channel from the official site]:<https://flutter.dev/docs/development/platform-integration/platform-channels>