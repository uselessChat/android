# Taking Photos Simply  

## Take a Photo with the Camera App
### Example: Get the Thumbnail  
Resource file with permissions codes  
```xml
<resources>
    <integer name="CAMERA">1</integer>
</resources>
```  

Resource file with result codes  
```xml
<resources>
    <integer name="IMAGE_CAPTURE">1</integer>
</resources>
```  

Interface delegate  
```java
public interface ICamera {
    void onProcess(Intent intent);
    void onReject(Intent intent);
}
```  

Domain logic  
```java
public class Camera {

    private Activity activity;
    private HashMap<String, Camera.Model> items;

    public Camera(@NonNull Activity activity) {
        this.activity = activity;
        this.items = new HashMap<>();
    }

    public void add(@NonNull String action, @NonNull ICamera delegate) {
        // e.g MediaStore.ACTION_IMAGE_CAPTURE = "android.media.action.IMAGE_CAPTURE";
        final String [] names = action.split("\\.");
        if (names.length == 0) return;

        final int resourceId = activity.getResources()
                .getIdentifier(names[names.length - 1], "integer", activity.getPackageName());
        final int code = activity.getResources().getInteger(resourceId);
        items.put(action, new Model(code, delegate));
    }

    public void onResult(final int requestCode, final int resultCode, Intent data) {
        Model cameraModel = null;
        for (final Model model : this.items.values()) {
            if (model.code == requestCode) {
                cameraModel = model;
                break;
            }
        }
        if (cameraModel == null) return;

        if (resultCode == RESULT_OK) {
            cameraModel.delegate.onProcess(data);
        } else {
            cameraModel.delegate.onReject(data);
        }
    }

    public void request(@NonNull String action) {
        request(action, null);
    }

    public void request(@NonNull String action, Uri output) {
        // e.g action = MediaStore.ACTION_IMAGE_CAPTURE
        Model model = items.get(action);
        if (model == null) { return; }

        Intent intent = new Intent(action);
        if (intent.resolveActivity(activity.getPackageManager()) != null) {
            if (output != null) { intent.putExtra(MediaStore.EXTRA_OUTPUT, output); }
            activity.startActivityForResult(intent, model.code);
        }
    }

    /** private **/
    private class Model {
        int code;
        ICamera delegate;

        Model(int code, @NonNull ICamera delegate) {
            this.code = code;
            this.delegate = delegate;
        }
    }
}
```  

Base or wraper activity  
```java
public class BaseActivity extends AppCompatActivity {

    private Permissions permissions;
    private Camera camera;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initCamera();
        initPermissions();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        camera.onResult(requestCode, resultCode, data);
    }

    @Override
    public void onRequestPermissionsResult(final int requestCode,
                                           @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        this.permissions.onRequest(requestCode, permissions, grantResults);
    }

    /** protected **/
    /** Camera **/
    protected void addCameraAction(@NonNull String action, @NonNull ICamera delegate) {
        this.camera.add(action, delegate);
    }

    protected void requestCameraAction(@NonNull String action) {
        this.camera.request(action);
    }

    /** Permissions **/
    protected void addPermission(@NonNull String permission, @NonNull IPermission delegate) {
        this.permissions.add(permission, delegate);
    }

    protected void requestPermission(@NonNull String permission) {
        this.permissions.request(permission);
    }

    /** private **/
    private void initCamera() {
        this.camera = new Camera(this);
    }

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
    }

    private void setActions() {
        // Simple example to Get the Thumbnail
        addCameraAction(MediaStore.ACTION_IMAGE_CAPTURE, new ICamera() {
            @Override
            public void onProcess(Intent intent) {
                Bundle extras = intent.getExtras();
                Bitmap imageBitmap = (Bitmap) extras.get("data");
                ImageView imageView = new ImageView(getApplicationContext());
                imageView.setImageBitmap(imageBitmap);
                Log.e("MainActivity", "Camera onProcess ACTION_IMAGE_CAPTURE");
            }

            @Override
            public void onReject(Intent intent) {
                // TODO:
            }
        });

        addPermission(Manifest.permission.CAMERA, new IPermission() {
            @Override
            public void onDenied() {
                Log.e("MainActivity", "CAMERA onDenied");
            }

            @Override
            public void onGranted() {
                Log.e("MainActivity", "CAMERA onGranted");
                requestCameraAction(MediaStore.ACTION_IMAGE_CAPTURE);
            }

            @Override
            public void onRationale() {
                Log.e("MainActivity", "CAMERA onRationale");
            }
        });
        // Type Button View
        findViewById(R.id.main_container).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                requestPermission(Manifest.permission.CAMERA);
            }
        });
    }
}
```  
