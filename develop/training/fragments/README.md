# Fragments

| < API 11 | >= API 11 |
| :------------- | :------------- |
| FragmentActivity | Activity |

| v4 | v7 appcompat |
| :------------- | :------------- |
| FragmentActivity | AppCompatActivity |

## Usgin XML
```xml
<fragment android:name="package.class" android:id:"@+id/fragment_id"/>
```

## Add/Replace a fragment
1. Activity layout must have a container(View) to insert the fragment
2. Get Fragment Manager
3. Create a Fragment Transaction (can be added to the back stack)
4. Add or Replace fragment on container(View)
5. Perform advanced fragment operations using [FragmentManager.BackStackEntry](https://developer.android.com/reference/android/support/v4/app/FragmentManager.BackStackEntry.html)

Example  
```java
// Create a Fragment Transaction
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
transaction.[add | replace](R.id.fragment_container, fragment);
// add the transaction to the back stack so the user can navigate back
transaction.addToBackStack(null);
// commit
transaction.commit();
```  

## Communicating with Other Fragments
1. Define an Interface
2. Implement the Interface
3. Deliver a Message to a Fragment

Example
```java
// Generic Fragment Interface
public interface IFragment<T> {
    void ensureDelegate();
    T emptyDelegate();
    void setDelegate(T delegate);
}
// Interface for fragment
public interface IEmptyFragment {
    void onActionName(Object object);
}
// Fragment
public class EmptyFragment extends Fragment implements IFragment<IEmptyFragment> {

    private IEmptyFragment delegate;

    public EmptyFragment() {
        // Required empty public constructor
    }

    public static EmptyFragment newInstance(IEmptyFragment delegate) {
        EmptyFragment emptyFragment = new EmptyFragment();
        emptyFragment.setDelegate(delegate);
        // emptyFragment.setArguments(new Bundle());
        return emptyFragment;
    }

    /* Life cycle */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Ensure that delegate is not null
        ensureDelegate();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_empty, container, false);
    }
    /* end Life cycle */

    /* IFragment*/
    @Override
    public void ensureDelegate() {
        if (this.delegate == null) {
            this.delegate = emptyDelegate();
        }
    }

    @Override
    public IEmptyFragment emptyDelegate() {
        return new IEmptyFragment() {
            @Override
            public void onActionName(Object object) {
                // ...
            }
        };
    }

    @Override
    public void setDelegate(IEmptyFragment delegate) {
        this.delegate = delegate != null ? delegate : emptyDelegate();
    }
    /* */
}
```












ddd
