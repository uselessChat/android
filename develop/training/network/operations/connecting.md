# Connecting to the Network
If not using okhttp/retrofit see [network-ops](https://developer.android.com/training/basics/network-ops/connecting.html)  
* Permissions  
* Design Secure Network Communication  
* Choose an HTTP Client  
* Network Operations on a Separate Thread  
  * [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html)
* Surviving Configuration Changes

## Permissions
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```  

## Design Secure Network Communication  
[Networking security tips](https://developer.android.com/training/articles/security-tips.html#Networking)
* Minimize the amount of sensitive or personal [user data](https://developer.android.com/training/articles/security-tips.html#UserData) that you transmit over the network.
* Send all network traffic from your app over [SSL](https://developer.android.com/training/articles/security-ssl.html).
* Consider creating a [network security configuration](https://developer.android.com/training/articles/security-config.html), which allows your app to trust custom CAs or restrict the set of system CAs that it trusts for secure communication.  

## Surviving Configuration Changes
```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    // Retain this Fragment across configuration changes in the host Activity.
    setRetainInstance(true);
}
```  
