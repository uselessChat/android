[Bluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth.html)
[Bluetooth Low Energy](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html)

Manifes  
```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-feature android:name="android.hardware.bluetooth_le"
        android:required="false" />
```  

Result codes
```xml
<resources>
    <!-- ... -->
    <integer name="ENABLE_BT">2</integer>
</resources>
```  

```java
public class Bluetooth {

    public static BluetoothAdapter getAdapter(Context context) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR2) {
            return BluetoothAdapter.getDefaultAdapter();
        }
        return ((BluetoothManager)context
                .getSystemService(Context.BLUETOOTH_SERVICE))
                .getAdapter();
    }

    public interface IBluetooth {
        void onProcess();
        void onReject(Intent intent);
        void onUnavailable();
        void onNotSupported();
    }

    private final Activity activity;
    private final int requestCode;
    private IBluetooth delegate;

    public Bluetooth(Activity activity) {
        this.activity = activity;
        requestCode = activity.getBaseContext().getResources()
                .getInteger(R.integer.ENABLE_BT);
    }

    public void call(@NonNull IBluetooth delegate) {
        this.delegate = delegate;
        final BluetoothAdapter bluetoothAdapter = getAdapter(activity.getBaseContext());
        if (bluetoothAdapter == null) { delegate.onNotSupported(); return; }
        if (!bluetoothAdapter.isEnabled()) {
            request(); // or delegate.onUnavailable()
            return;
        }
        delegate.onProcess();
    }

    public void onResult(final int requestCode, final int resultCode, Intent data) {
        if (this.requestCode != requestCode) return;

        if (resultCode == RESULT_OK) {
            this.delegate.onProcess();
        } else {
            this.delegate.onReject(data);
        }
    }

    private void request() {
        final Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
        activity.startActivityForResult(intent, this.requestCode);
    }

    public static class LowEnergy {
        public static boolean isSupported(Context context) {
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR2)
                return false;
            return context.getPackageManager()
                    .hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE);
        }

        private final Context context;

        public LowEnergy(Context context) {
            this.context = context;
        }

        public void call(IBluetooth delegate) {
            if (!isSupported(context)) { delegate.onNotSupported(); return; }
            delegate.onProcess();
        }
    }
}

public interface IBluetoothBroadcast {
    void onReceive(Context context, Intent intent);
}

public abstract class BluetoothBroadcast implements IBluetoothBroadcast {

    private final Context context;
    private Receiver receiver;
    private final IntentFilter intentFilter;
    private Bluetooth.IBluetooth delegate;

    public BluetoothBroadcast(Context context) {
        this.context = context;
        intentFilter = new IntentFilter(BluetoothAdapter.ACTION_STATE_CHANGED);
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // Extra fields are:
        // STATE_TURNING_ON, STATE_ON, STATE_TURNING_OFF, STATE_OFF
        final int state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, 0);
        if (BluetoothAdapter.STATE_ON != state) return;
        delegate.onProcess();
    }

    public abstract void call(@NonNull Bluetooth.IBluetooth delegate);

    public void run(@NonNull Bluetooth.IBluetooth delegate) {
        this.delegate = delegate;
        run();
    }

    public void stop() {
        if (receiver == null) return;

        // Unregisters BroadcastReceiver when app is destroyed.
        context.unregisterReceiver(receiver);
    }

    private void call() {
        call(delegate);
    }

    private void run() {
        if (delegate == null) return;
        start();
        delegate.onUnavailable();
    }

    private void start() {
        if (receiver == null) return;

        // Registers BroadcastReceiver to track network connection changes.
        receiver = new Receiver(this);
        context.registerReceiver(receiver, intentFilter);
    }

    private class Receiver extends BroadcastReceiver {
        private final IBluetoothBroadcast delegate;

        public Receiver(IBluetoothBroadcast delegate) {
            this.delegate = delegate;
        }

        @Override
        public void onReceive(Context context, Intent intent) {
            delegate.onReceive(context, intent);
        }
    }
}
```
