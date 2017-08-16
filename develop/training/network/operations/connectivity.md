# [ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager.html)
# [NetworkInfo](https://developer.android.com/reference/android/net/NetworkInfo.html)  
# [Broadcasts](https://developer.android.com/guide/components/broadcasts.html)  
1. Monitor network connections (Wi-Fi, GPRS, UMTS, etc.)
2. Send broadcast intents when network connectivity changes
3. Attempt to "fail over" to another network when connectivity to a network is lost
4. Provide an API that allows applications to query the coarse-grained or fine-grained state of the available networks
5. Provide an API that allows applications to request and select networks for their data traffic.  

# Network Operations

Base class and interface  
```java
// Interface
public interface INetworkOperation {
    void onProcess();
    void onReject();
    void onUnavailable();
}
// Abstract class
public abstract class NetworkOperation {

    protected final Context context;
    protected final ConnectivityManager connectivityManager;

    public NetworkOperation(Context context) {
        this.context = context;
        connectivityManager = (ConnectivityManager)context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
    }

    public boolean isAvailable() {
        boolean isConnected = false;

        for (Integer type : types()) {
            final NetworkInfo networkInfo =
                    connectivityManager.getNetworkInfo(type);
            isConnected = networkInfo != null && networkInfo.isConnected();
            if (isConnected) break;
        }
        return isConnected;
    }

    public boolean isTypeAcceptable(final int networkType) {
        boolean isAccepted = false;
        for (Integer type : types()) {
            if (networkType == type) {
                isAccepted = true;
                break;
            }
        }
        return isAccepted;
    }

    /** protected **/
    protected abstract void call(INetworkOperation delegate);
    protected abstract List<Integer> types();
}
```  

Activity Example with a simple operation  
```java
public class NetworkActivity extends AppCompatActivity {

    private Toolbar toolbar;
    private FloatingActionButton floatingActionButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_basic);
        setViews();

        // Operation
        new NetworkActivityOperation(getApplicationContext()).call(new INetworkOperation() {
            @Override
            public void onProcess() {
                Snackbar.make(floatingActionButton, "Message", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }

            @Override
            public void onReject() {
                // ...
            }

            @Override
            public void onUnavailable() {
                // ...
            }
        });
    }

    /** private **/

    private void setViews() {
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        floatingActionButton = (FloatingActionButton) findViewById(R.id.fab);
    }

    private class NetworkActivityOperation extends NetworkOperation {
        public NetworkActivityOperation(Context context) {
            super(context);
        }

        @Override
        protected void call(INetworkOperation delegate) {
            if (!isAvailable()) {
                delegate.onUnavailable();
                return;
            }
            // TODO: some async operations
            delegate.onProcess(); // delegate.onReject();
        }

        @Override
        protected List<Integer> types() {
            return Collections.singletonList(ConnectivityManager.TYPE_WIFI);
        }
    }
}
```  

# Network Operations with Broadcasts  

Base class and Interface  
```java
// Interface
// Wraper for method onReceive from BroadcastReceiver
public interface INetworkReceiverOperation {
    void onReceive(Context context, Intent intent);
}
// Abstract class
public abstract class NetworkReceiverOperation extends NetworkOperation
        implements INetworkReceiverOperation {

    private INetworkOperation delegate;
    private Receiver receiver;
    private final IntentFilter intentFilter;

    public NetworkReceiverOperation(@NonNull Context context) {
        super(context);
        this.receiver = new Receiver(this);
        // A change in network connectivity has occurred
        this.intentFilter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // Network type which triggered a CONNECTIVITY_ACTION broadcast.
        final int networkType =
                intent.getIntExtra(ConnectivityManager.EXTRA_NETWORK_TYPE, 0);
        // Verify that the networkType match any in types by user
        if (!isTypeAcceptable(networkType)) return;
        stop();
        run();
    }

    public void run(@NonNull INetworkOperation delegate) {
        this.delegate = delegate;
        run();
    }

    public void stop() {
        if (this.receiver == null) return;
        // Unregisters BroadcastReceiver when app is destroyed.
        context.unregisterReceiver(receiver);
    }

    /** private **/

    private void call() {
        call(delegate);
    }

    private void run() {
        if (delegate == null) return;

        if (isAvailable()) { call(); return; }
        start();
        delegate.onUnavailable();
    }

    private void start() {
        if (this.receiver == null) return;

        // Registers BroadcastReceiver to track network connection changes.
        this.receiver = new Receiver(this);
        context.registerReceiver(receiver, intentFilter);
    }

    private class Receiver extends BroadcastReceiver {
        private final NetworkReceiverOperation delegate;

        public Receiver(NetworkReceiverOperation delegate) {
            this.delegate = delegate;
        }

        @Override
        public void onReceive(Context context, Intent intent) {
            delegate.onReceive(context, intent);
        }
    }
}
```  

Activity Example with a simple operation and waiting for a broadcast  
```java
public class NetworkReceiverActivity extends AppCompatActivity {

    private Toolbar toolbar;
    private FloatingActionButton floatingActionButton;
    private NetworkReceiverActivityOperation operation;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_basic);
        setViews();
        setLocalData();

        // Operation
        // Send message "run" instead of "call", we want to listen network changes
        // Register a receiver if no network available from within types
        operation.run(new INetworkOperation() {
            @Override
            public void onProcess() {
                Snackbar.make(floatingActionButton, "Message", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }

            @Override
            public void onReject() {
                // ...
            }

            @Override
            public void onUnavailable() {
                // ...
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Stop listening network changes
        operation.stop();
    }

    /** private **/

    private void setLocalData() {
        operation = new NetworkReceiverActivityOperation(getBaseContext());
    }

    private void setViews() {
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        floatingActionButton = (FloatingActionButton) findViewById(R.id.fab);
    }
    // Operation definition
    private class NetworkReceiverActivityOperation extends NetworkReceiverOperation {
        public NetworkReceiverActivityOperation(Context context) {
            super(context);
        }

        @Override
        protected void call(INetworkOperation delegate) {
            if (!isAvailable()) {
                delegate.onUnavailable();
                return;
            }
            // TODO: some async operations
            delegate.onProcess(); // delegate.onReject();
        }

        @Override
        protected List<Integer> types() {
            return Collections.singletonList(ConnectivityManager.TYPE_WIFI);
        }
    }
}
```  
