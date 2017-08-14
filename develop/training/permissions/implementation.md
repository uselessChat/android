Resource file with permissions codes  

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <integer name="WRITE_CALENDAR">1</integer>
</resources>
```  

Interface delegate  
```java
public interface IPermission {
    void onDenied();
    void onGranted();
    void onRationale();
}
```  

Domain logic  
```java
public class Permissions {

    private Activity activity;
    private HashMap<String, Model> items;

    public Permissions(Activity activity) {
        this.activity = activity;
        this.items = new HashMap<>();
    }

    public void add(String permission, IPermission delegate) {
        final String [] names = permission.split("\\.");
        if (names.length == 0) return;

        final int resourceId = activity.getResources()
                .getIdentifier(names[names.length - 1], "integer", activity.getPackageName());
        final int code = activity.getResources().getInteger(resourceId);
        items.put(permission, new Model(code, delegate));
    }

    public void onRequest(final int requestCode,
                          @NonNull String[] permissions,
                          @NonNull int[] grantResults) {
        final boolean result = grantResults.length > 0 &&
                grantResults[0] == PackageManager.PERMISSION_GRANTED;
        Model permissionModel = null;

        for (final Model model : this.items.values()) {
            if (model.code == requestCode) {
                permissionModel = model;
                break;
            }
        }
        if (permissionModel == null) return;

        if (result) {
            permissionModel.delegate.onGranted();
        } else {
            permissionModel.delegate.onDenied();
        }
    }

    public void request(@NonNull final String permission) {
        Model model = items.get(permission);
        if (model == null) return;

        IPermission delegate = model.delegate;
        int checkPermission =
                ContextCompat.checkSelfPermission(activity.getApplicationContext(), permission);
        if (checkPermission == PackageManager.PERMISSION_GRANTED) {
            delegate.onGranted();
            return;
        }
        // Should we show an explanation?
        boolean showRationale = ActivityCompat.shouldShowRequestPermissionRationale(activity, permission);
        if (showRationale) {
            // Show an explanation to the user *asynchronously* -- don't block
            // this thread waiting for the user's response! After the user
            // sees the explanation, try again to request the permission.
            delegate.onRationale();
            return;
        }
        // No explanation needed, we can request the permission.
        ActivityCompat.requestPermissions(activity, new String[]{permission}, model.code);
    }

    /** private **/
    private class Model {
        int code;
        IPermission delegate;

        Model(int code, @NonNull IPermission delegate) {
            this.code = code;
            this.delegate = delegate;
        }
    }
}
```  

Base or wraper activity  
```java
/**
 * Super Activity with our domain logic
 * This is necessary because of activity callback onRequestPermissionsResult
 * TODO: still need some analysis
**/
public class BaseActivity extends AppCompatActivity {

    private Permissions permissions;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initPermissions();
    }

    @Override
    public void onRequestPermissionsResult(final int requestCode,
                                           @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        this.permissions.onRequest(requestCode, permissions, grantResults);
    }

    /** protected **/
    protected void addPermission(@NonNull String permission, @NonNull IPermission delegate) {
        this.permissions.add(permission, delegate);
    }

    protected void requestPermission(@NonNull String permission) {
        this.permissions.request(permission);
    }

    /** private **/
    private void initPermissions() {
        this.permissions = new Permissions(this);
    }
}
```

Example  
```java
public class MainActivity extends BaseActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        setActions();
    }

    @Override
    protected void onStart() {
        super.onStart();
        requestPermission(Manifest.permission.WRITE_CALENDAR);
    }

    private void setActions() {
        addPermission(Manifest.permission.WRITE_CALENDAR, new IPermission() {
            @Override
            public void onDenied() {
                Log.e("MainActivity", "onDenied");
            }

            @Override
            public void onGranted() {
                Log.e("MainActivity", "onGranted");
            }

            @Override
            public void onRationale() {
                Log.e("MainActivity", "onRationale");
            }
        });
    }
}
```
