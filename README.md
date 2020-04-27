1)flutter create map_app2
2)cd map_app
3)flutter run
4)cd map_app2
5)git clone githttps://github.com/stereci/Offline-MapBox-App.git
6)Then update your pubspec.yaml to reference the directory where the project was cloned.
dependencies:
  ...
  mapbox_gl:
    path: ./flutter-mapbox-gl
7)flutter pub get

Android Changes
8)Add your API token to the AndroidManifest -> android/app/src/main/AndroidManifest.xml
<application
    android:name="io.flutter.app.FlutterApplication"
    android:label="map_app"
    android:icon="@mipmap/ic_launcher">    <meta-data android:name="com.mapbox.token"
        android:value="YOUR API KEY" />    <activity
    ...
</application>
9)You will also need to migrate the ./android directory to use androidx by adding the following two lines to ./android/gradle.properties
android.useAndroidX=true
android.enableJetifier=true

iOS Changes
10)ios/Runner/Info.plist
<plist version="1.0">
<dict>
    ...
    <key>MGLMapboxAccessToken</key>
    <string>YOUR API KEY</string
    <key>io.flutter.embedded_views_preview</key>
    <true/>
</dict>
</plist>
11)ios/Podfile
Uncomment the platform line to target iOS 9 required by the mapbox_gl plugin and add the use frameworks! line.
# Uncomment this line to define a global platform for your project
platform :ios, '9.0'
use_frameworks!

THE CODE -> main.dart(last stage)

import 'package:flutter/material.dart';
import 'package:mapbox_gl/mapbox_gl.dart';

import 'dart:async';
import 'dart:io';
import 'package:path/path.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: MapWidget());
  }
}

class MapWidget extends StatefulWidget {
  @override
  _MapWidgetState createState() => _MapWidgetState();
}

class _MapWidgetState extends State<MapWidget> {
  final CameraPosition _kInitialPosition;
  final CameraTargetBounds _cameraTargetBounds;
  static double defaultZoom = 12.0;

  var _tilesLoaded = false;

  CameraPosition _position;
  MapboxMapController mapController;
  bool _isMoving = false;
  bool _compassEnabled = true;
  MinMaxZoomPreference _minMaxZoomPreference =
  const MinMaxZoomPreference(12.0, 18.0);
  String _styleString = "mapbox://styles/mapbox/streets-v11";
  bool _rotateGesturesEnabled = true;
  bool _scrollGesturesEnabled = true;
  bool _tiltGesturesEnabled = false;
  bool _zoomGesturesEnabled = true;
  bool _myLocationEnabled = false;
  MyLocationTrackingMode _myLocationTrackingMode = MyLocationTrackingMode.None;

  _MapWidgetState._(
      this._kInitialPosition, this._position, this._cameraTargetBounds);

  static CameraPosition _getCameraPosition() {
    final latLng = LatLng(41.0082, 28.9784);
    return CameraPosition(
      target: latLng,
      zoom: defaultZoom,
    );
  }
  @override
  initState() {
    super.initState();
    _copyTilesIntoPlace();
  }  _copyTilesIntoPlace() async {
    try {
      await installOfflineMapTiles(join("assets", "cache.db"));
    } catch (err) {
      print(err);
    }    setState(() {
      this._tilesLoaded = true;
    });
  }

  factory _MapWidgetState() {
    CameraPosition cameraPosition = _getCameraPosition();

    final cityBounds = LatLngBounds(
      southwest: LatLng(40.9682, 29.0384),
      northeast: LatLng(41.0582, 28.9184),
    );

    return _MapWidgetState._(
        cameraPosition, cameraPosition, CameraTargetBounds(cityBounds));
  }

  void _onMapChanged() {
    setState(() {
      _extractMapInfo();
    });
  }

  @override
  void dispose() {
    if (mapController != null) {
      mapController.removeListener(_onMapChanged);
    }
    super.dispose();
  }

  void _extractMapInfo() {
    _position = mapController.cameraPosition;
    _isMoving = mapController.isCameraMoving;
  }

  @override
  Widget build(BuildContext context) {
    if (this._tilesLoaded) {
      return Container(
        child: _buildMapBox(context),
      );
    } else {
      return Center(
        child: new CircularProgressIndicator(),
      );
    }
  }

  MapboxMap _buildMapBox(BuildContext context) {
    return MapboxMap(
        onMapCreated: onMapCreated,
        initialCameraPosition: this._kInitialPosition,
        trackCameraPosition: true,
        compassEnabled: _compassEnabled,
        cameraTargetBounds: _cameraTargetBounds,
        minMaxZoomPreference: _minMaxZoomPreference,
        styleString: _styleString,
        rotateGesturesEnabled: _rotateGesturesEnabled,
        scrollGesturesEnabled: _scrollGesturesEnabled,
        tiltGesturesEnabled: _tiltGesturesEnabled,
        zoomGesturesEnabled: _zoomGesturesEnabled,
        myLocationEnabled: _myLocationEnabled,
        myLocationTrackingMode: _myLocationTrackingMode,
        onCameraTrackingDismissed: () {
          this.setState(() {
            _myLocationTrackingMode = MyLocationTrackingMode.None;
          });
        });
  }

  void onMapCreated(MapboxMapController controller) {
    mapController = controller;
    mapController.addListener(_onMapChanged);
    _extractMapInfo();
    setState(() {});
  }
}

Adding the tiles
With the map tiles downloaded we add them to our assets directory. The db files needs to be named cache.db.
11)mkdir -p map_app/assets/
12)cp mapbox-gl-native/mapcache.db map_app/assets/cache.db
13)Update your pubspec.yaml to include the cache.db file into your app
assets:
    - assets/cache.db
14)Run in Airplane mode
