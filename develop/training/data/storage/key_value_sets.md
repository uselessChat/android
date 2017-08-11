# Storage
Small collection of key-values with [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html).  
```java
public class Storage {
    private static final String FILE_KEY = null; //= BuildConfig.PREFERENCE_FILE_KEY;
    private SharedPreferences sharedPreferences;

    public Storage(Context context) {
        this.sharedPreferences = context.getSharedPreferences(FILE_KEY,
                Context.MODE_PRIVATE);
    }

    public void clear() {
        for (String key : sharedPreferences.getAll().keySet()) {
            setValue(key, null);
        }
    }

    public String getValue(String key) {
        return sharedPreferences.getString(key, null);
    }

    public void setValue(String key, String value) {
        sharedPreferences.edit().putString(key, value).apply();
    }
}
```


ddd
