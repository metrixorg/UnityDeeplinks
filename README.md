# UnityDeeplinks
A set of tools for Unity to allow handling deeplink activation from within Unity scripts, for Android and iOS, including iOS Universal Links.

### Known Issues/Limitations
* Check out the repo's *issues* section

### Disclaimer
This is NOT a TROPHiT SDK - this repo is an open-source contribution to developers for handling deeplink activations in a unified way for both iOS and Android. It can be used independently and regardless of TROPHiT services in order to intercept deeplinks for whatever purpose. If you are looking for info about TROPHiT integration modules, visit the [TROPHiT Help Center](https://trophit.zendesk.com/hc/en-us/articles/200865062-How-do-I-integrate-TROPHiT-)

# Usage
#### Example: Track Deeplinks with Metrix
* Tested with [Metrix Unity Plugin](https://storage.backtory.com/metricx/sdk-unity/MetrixSDK-v0.11.0.unitypackage) v0.11.0
* Assuming you already integrated the Metrix Plugin, just implement `onDeeplink` in *UnityDeeplinks.cs* as follows:

 ```cs
 public void onDeeplink(string deeplink) {
    Metrix.AppWillOpenUrl(deeplink);
 }
 ```

# Integration
* Clone/download the repository
* Copy the entire UnityDeeplinks folder into your Unity project Assets folder
* Attach the *Assets/UnityDeeplinks/UnityDeeplinks.cs* script to an empty *UnityDeeplinks* game object

## Android
Subclass the default *UnityPlayerActivity* in order to add deeplink-handling code that marshals deeplinks into your Unity script:

* Replace the default UnityPlayerActivity in your Assets/Plugins/Android/AndroidManifest.xml with com.trophit.MyUnityPlayerActivity:

 ```xml
 <!--
 <activity android:name="com.unity3d.player.UnityPlayerActivity" ...
 -->
 <activity android:name="com.trophit.MyUnityPlayerActivity" ...
 ```

* Add the following inside the same *activity* tag, assuming your deeplink URL scheme is myapp://

 ```xml
 <intent-filter>
     <action android:name="android.intent.action.VIEW" />
     <category android:name="android.intent.category.DEFAULT" />
     <category android:name="android.intent.category.BROWSABLE" />
     <data android:scheme="myapp" />
 </intent-filter>
 ```

* Notes:
 * If you already subclassed your Unity activity, merge the code from within *MyUnityPlayerActivity* into your existing subclass
 * Optional: by default, *MyUnityPlayerActivity* calls a Unity script method `onDeeplink` on a game object called *UnityDeeplinks*. If you wish to change the name of the object or method, you should edit *Assets/UnityDeeplinks/Android/MyUnityPlayerActivity.java*, change the values of the `gameObject` and/or `deeplinkMethod` static properties and rebuild the *UnityDeeplinks.jar* file as instructed below

### Why not handle deeplinks in a second activity?
Some might suggest having a "side-activity" e.g. *MyDeeplinkActivity* to handle the deeplink and start the main Unity player activity. This way, the main Unity player activity remains clean of "outside code", right? Wrong. Consider the Unity app is currently not running. Then:
* A deeplink gets activated
* MyDeeplinkActivity starts
* Tries to access the UnityPlayer object in order to send a message to a Unity script with the deeplink information
* At this point, since the Unity native libraries are not yet initialized, the call would fail with the error:
 ```
 Native libraries not loaded - dropping message for ...
 ```
Bottom line: you need the Unity player activity initialized in order to call Unity functions from native code. The only way to handle the scenario above would be to have the Unity player activity itself handle the deeplink. Unity will make sure it's initialized prior to the call.

### Building the UnityDeeplinks.jar file
Only perform this step if you made changes to any .java file under *Assets/UnityDeeplinks/Android/* or would like to rebuild it using an updated version of Unity classes, Android SDK, JDK and so on.

#### Prerequisites
* Go to your Unity => Preferences => External tools menu and find your Android SDK and JDK home folders
* Edit *UnityDeeplinks/Android/bulid_jar.sh*
* Ensure ANDROID_SDK_ROOT points to your Android SDK root folder
* Ensure JDK_HOME points to your JDK root folder
* Ensure UNITY_LIBS points to the Unity classes.jar file in your development environment

#### Build instructions
Run the build script:
```
cd MY_UNITY_PROJECT_ROOT/Assets/UnityDeeplinks/Android
./build_jar.sh
```

Example output:
```
Compiling ...
/Library/Java/JavaVirtualMachines/jdk1.7.0_51.jdk/Contents/Home/jre/lib
/Applications/Unity/PlaybackEngines/AndroidPlayer/Variations/mono/Release/Classes/classes.jar:/kankado/dev/tools/android-sdk-macosx/platforms/android-23/android.jar
Creating jar file...
adding: com/(in = 0) (out= 0)(stored 0%)
adding: com/trophit/(in = 0) (out= 0)(stored 0%)
adding: com/trophit/DeeplinkActivity.class(in = 2368) (out= 1237)(deflated 47%)
adding: com/trophit/MyUnityPlayerActivity.class(in = 1504) (out= 789)(deflated 47%)
```

This creates/updates a *UnityDeeplinks.jar* file under your Unity project's Assets/UnityDeeplinks folder.

Finally, continue to build and test your Unity project as usual in order for any jar changes to take effect

## iOS
UnityDeeplinks implements a native plugin for iOS, initialized by *Assets/UnityDeeplinks/UnityDeeplinks.cs*. The plugin listens for URL/Univeral Link activations and relayes them to the Unity script for processing. It, too, uses a similar approach as the one used for Android: the main Unity app controller gets subclassed.

Also, like in the Android case, if the app is currently not running, we can't simply have the native low-level deeplinking code make calls to Unity scripts until the Unity libraries are initialized. Therefore, we store the deeplink in a variable, wait for the app to initialize the plugin (an indication that the Unity native libraries are ready), and then send the stored deeplink to the Unity script.

* To support URL schemes, go to your Unity project's Build => iOS Player Settings => Other Settings => Supported URL Schemes and set:
 * Size: 1
 * Element 0: your URL scheme, e.g. myapp
* To support Universal Links, set them up as per [their specification](https://developer.apple.com/library/content/documentation/General/Conceptual/AppSearch/UniversalLinks.html). *Note:* Universal Link settings in your XCode project are not removed during Unity rebuilds, unless you use the *Replace* option and overwrite your XCode project

## Testing

* Prepare a dummy web page that is accessible by your mobile device:

 ```xml
 <body>
 <a href="myapp://?a=b">deeplink test</a>
 </body>
 ```

* Open the web page on the device browser and click the deeplink
* The Unity app should open and the onDeeplink Unity script method should be invoked, performing whatever it is you designed it to perform