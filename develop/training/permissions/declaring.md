# Permissions
[Platform Versions](https://developer.android.com/about/dashboards/index.html#Platform)
[System Permissions](https://developer.android.com/guide/topics/permissions/requesting.html)

Categories
1. Normal
2. Dangerous

## Declaring
| API <= 22 | API > 22 |
| :------------- | :------------- |
| Installation | Run time |
| Normal | Normal & Dangerous |

Manifest  
Child of the top-level <manifest> element    
```xml
<uses-permission android:name="string" android:maxSdkVersion="integer" />
```  
