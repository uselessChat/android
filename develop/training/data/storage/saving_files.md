# Storage
A [File](https://developer.android.com/reference/java/io/File.html) object is suited to reading or writing large amounts of data in start-to-finish order without skipping around.  

## External Storage
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <!-- if read the external storage -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
</manifest>
```  

Example  
```java
public interface IStorage {
    void onProcess();
    void onReject();
}
public class StorageExternal {

    public static void readable(IStorage delegate) {
        String state = Environment.getExternalStorageState();
        if (Environment.MEDIA_MOUNTED.equals(state) ||
                Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
            delegate.onProcess();
        } else {
            delegate.onReject();
        }
    }

    public static void writeable(IStorage delegate) {
        String state = Environment.getExternalStorageState();
        if (Environment.MEDIA_MOUNTED.equals(state)) {
            delegate.onProcess();
        } else {
            delegate.onReject();
        }
    }

    public static class Public {
        /**
         * Save public files on the external storage
         * @param type Check on Environment.DIRECTORY_[NAME]
         */
        public static File getDirectory(String name, String type) {
            File file = new File(Environment.getExternalStoragePublicDirectory(type), name);
            if (!file.mkdirs()) {
                Log.e("TAG", "Directory not created");
            }
            return file;
        }
    }

    public static class Private {
        /**
         * Save files that are private to your app
         * @param type Check on Environment.DIRECTORY_[NAME]
         */
        public static File getDirectory(Context context, String name, String type) {
            // Get the directory for the app's private pictures directory.
            File file = new File(context.getExternalFilesDir(type), name);
            if (!file.mkdirs()) {
                Log.e("TAG", "Directory not created");
            }
            return file;
        }
    }
}
```

## Internal Storage
1. context#getFilesDir -> File
2. context#getCacheDir -> File
3. context#openFileOutput(filename, Context.MODE_PRIVATE) -> FileOutputStream

### Saving  
Example  
```java
File file = new File(context.getFilesDir(), filename);
```  

### Deleting
Example  
```java
context.deleteFile(fileName);
```  
