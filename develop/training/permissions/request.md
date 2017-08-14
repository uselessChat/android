# Permissions

## Requesting
Android Support Library  
1. Check for permissions => ContextCompat.checkSelfPermission()
  - PackageManager.PERMISSION_GRANTED
  - PackageManager.PERMISSION_DENIED
2. Request Permissions
  - Explain why the app needs permissions
    => ActivityCompat.shouldShowRequestPermissionRationale()
  - Request the permissions you need
    => ActivityCompat.requestPermissions()
     - ActivityCompat.OnRequestPermissionsResultCallback
  - Handle the permissions request response  
