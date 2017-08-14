## Request Camera Permission  
```xml
<manifest ... >
  <uses-feature android:name="android.hardware.camera"
      android:required="true" />
  <uses-permission android:name="android.permission.CAMERA" />
  ...
</manifest>
```  

If required false then check for
```java
hasSystemFeature(PackageManager.FEATURE_CAMERA)
```  
