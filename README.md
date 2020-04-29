## Getting Started

* flutter create mapapp
* cd mapapp
* flutter run
* cd Offline-MapBox-App
* git clone githttps: ->github.com/stereci/Offline-MapBox-App.git
* Then update your pubspec.yaml to reference the directory where the project was cloned.
```
    dependencies:
     ...
    mapbox_gl:
       path: ./Offline-MapBox-App
```
* flutter pub get

**Android Changes**
* Add your API token to the AndroidManifest -> android/app/src/main/AndroidManifest.xml
```
<application
    android:name="io.flutter.app.FlutterApplication"
    android:label="map_app"
    android:icon="@mipmap/ic_launcher">    <meta-data android:name="com.mapbox.token"
        android:value="YOUR API KEY" />    <activity
    ...
</application>
```
* You will also need to migrate the ./android directory to use androidx by adding the following two lines to ./android/gradle.properties
* android.useAndroidX=true
* android.enableJetifier=true

**iOS Changes**
* ios/Runner/Info.plist
```
<plist version="1.0">
<dict>
    ...
    <key>MGLMapboxAccessToken</key>
    <string>YOUR API KEY</string
    <key>io.flutter.embedded_views_preview</key>
    <true/>
</dict>
</plist>
```
* ios/Podfile 
``
Uncomment the platform line to target iOS 9 required by the mapbox_gl plugin and add the use frameworks! line.
``
```
# Uncomment this line to define a global platform for your project
platform :ios, '9.0'
use_frameworks!
```
* flutter run //until here, if u run your app, it will open an online mapbox app, for offline support follow the steps

**Build MapBox tools**
* git clone https://github.com/mapbox/mapbox-gl-native.git
* cd mapbox-gl-native
* make

**Adding the tiles**
With the map tiles downloaded we add them to our assets directory. The db files needs to be named cache.db.
```
mkdir -p map_app/assets/
cp mapbox-gl-native/mapcache.db map_app/assets/cache.db
Update your pubspec.yaml to include the cache.db file into your app
    assets:
        - assets/cache.db
```
* Run in Airplane mode

