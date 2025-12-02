# Android Bridge Implementation

Comprehensive guide to implementing React Native native modules for Android using Java and Kotlin.

## Contents
- Java implementation
- Kotlin implementation
- Threading and performance
- Lifecycle management
- Advanced patterns

## Java Implementation

### Basic Module Setup

**Module class:**
```java
package com.mymodule;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

public class MyModule extends ReactContextBaseJavaModule {
    private final ReactApplicationContext reactContext;

    public MyModule(ReactApplicationContext context) {
        super(context);
        this.reactContext = context;
    }

    @Override
    public String getName() {
        return "MyModule";  // Used in JS: NativeModules.MyModule
    }
}
```

**Package class:**
```java
package com.mymodule;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MyModulePackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new MyModule(reactContext));
        return modules;
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

**Register in MainApplication.java:**
```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.asList(
        new MainReactPackage(),
        new MyModulePackage()  // Add your package
    );
}
```

### Method Export

**Synchronous methods:**
```java
// Returns immediately (blocks JS thread)
@ReactMethod(isBlockingSynchronousMethod = true)
public String getValue() {
    return "value";
}

@ReactMethod(isBlockingSynchronousMethod = true)
public int add(int a, int b) {
    return a + b;
}
```

**Asynchronous with Promise:**
```java
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;

@ReactMethod
public void fetchData(Promise promise) {
    try {
        // Perform operation
        WritableMap result = Arguments.createMap();
        result.putString("data", "value");
        result.putInt("count", 42);
        
        promise.resolve(result);
    } catch (Exception e) {
        promise.reject("FETCH_ERROR", "Failed to fetch data", e);
    }
}
```

**Asynchronous with Callback:**
```java
import com.facebook.react.bridge.Callback;

@ReactMethod
public void processData(String input, Callback callback) {
    try {
        String result = process(input);
        // Callback format: error, result
        callback.invoke(null, result);
    } catch (Exception e) {
        callback.invoke(e.getMessage(), null);
    }
}
```

### Event Emitters

```java
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.modules.core.DeviceEventManagerModule;

public class MyModule extends ReactContextBaseJavaModule {
    
    private void sendEvent(String eventName, WritableMap params) {
        reactContext
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
            .emit(eventName, params);
    }
    
    @ReactMethod
    public void startMonitoring() {
        // Start monitoring and send events
        WritableMap data = Arguments.createMap();
        data.putString("status", "active");
        sendEvent("onUpdate", data);
    }
}
```

### Data Type Conversion

**JavaScript to Java:**
```java
import com.facebook.react.bridge.ReadableMap;
import com.facebook.react.bridge.ReadableArray;

@ReactMethod
public void handleData(ReadableMap data) {
    // String
    String name = data.getString("name");
    
    // Number
    int age = data.getInt("age");
    double price = data.getDouble("price");
    
    // Boolean
    boolean isActive = data.getBoolean("active");
    
    // Array
    ReadableArray items = data.getArray("items");
    for (int i = 0; i < items.size(); i++) {
        String item = items.getString(i);
    }
    
    // Nested object
    ReadableMap settings = data.getMap("settings");
    
    // Check if key exists
    if (data.hasKey("optional")) {
        String optional = data.getString("optional");
    }
}
```

**Java to JavaScript:**
```java
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableArray;
import com.facebook.react.bridge.Arguments;

@ReactMethod
public void getData(Promise promise) {
    WritableMap result = Arguments.createMap();
    
    // Primitives
    result.putString("string", "value");
    result.putInt("number", 42);
    result.putDouble("float", 3.14);
    result.putBoolean("boolean", true);
    result.putNull("null");
    
    // Array
    WritableArray array = Arguments.createArray();
    array.pushString("a");
    array.pushString("b");
    array.pushInt(1);
    result.putArray("array", array);
    
    // Nested object
    WritableMap nested = Arguments.createMap();
    nested.putString("key", "value");
    result.putMap("nested", nested);
    
    promise.resolve(result);
}
```

## Kotlin Implementation

### Basic Module

```kotlin
package com.mymodule

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import com.facebook.react.bridge.Promise

class MyModule(reactContext: ReactApplicationContext) : 
    ReactContextBaseJavaModule(reactContext) {
    
    override fun getName() = "MyModule"
    
    // Synchronous method
    @ReactMethod(isBlockingSynchronousMethod = true)
    fun getValue(): String {
        return "value"
    }
    
    // Async with Promise
    @ReactMethod
    fun fetchData(promise: Promise) {
        try {
            val result = Arguments.createMap().apply {
                putString("data", "value")
                putInt("count", 42)
            }
            promise.resolve(result)
        } catch (e: Exception) {
            promise.reject("FETCH_ERROR", "Failed to fetch data", e)
        }
    }
    
    // Async with callback
    @ReactMethod
    fun processData(input: String, callback: Callback) {
        try {
            val result = process(input)
            callback.invoke(null, result)
        } catch (e: Exception) {
            callback.invoke(e.message, null)
        }
    }
}
```

### Event Emitters in Kotlin

```kotlin
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.Arguments
import com.facebook.react.modules.core.DeviceEventManagerModule

class MyModule(private val reactContext: ReactApplicationContext) : 
    ReactContextBaseJavaModule(reactContext) {
    
    override fun getName() = "MyModule"
    
    private fun sendEvent(eventName: String, params: WritableMap?) {
        reactContext
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
            .emit(eventName, params)
    }
    
    @ReactMethod
    fun startMonitoring() {
        val data = Arguments.createMap().apply {
            putString("status", "active")
            putLong("timestamp", System.currentTimeMillis())
        }
        sendEvent("onUpdate", data)
    }
}
```

### Kotlin Extensions

```kotlin
// Helpful extensions for cleaner code
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.Arguments

fun createMap(builder: WritableMap.() -> Unit): WritableMap {
    return Arguments.createMap().apply(builder)
}

// Usage
@ReactMethod
fun getData(promise: Promise) {
    val result = createMap {
        putString("name", "John")
        putInt("age", 30)
        putBoolean("active", true)
    }
    promise.resolve(result)
}
```

## Threading and Performance

### Background Thread Execution

```java
import android.os.Handler;
import android.os.Looper;

@ReactMethod
public void heavyOperation(Promise promise) {
    // Execute on background thread
    new Thread(() -> {
        try {
            String result = performHeavyComputation();
            
            // Return to main thread for promise resolution
            new Handler(Looper.getMainLooper()).post(() -> {
                promise.resolve(result);
            });
        } catch (Exception e) {
            new Handler(Looper.getMainLooper()).post(() -> {
                promise.reject("ERROR", e);
            });
        }
    }).start();
}
```

**Kotlin coroutines:**
```kotlin
import kotlinx.coroutines.*

@ReactMethod
fun heavyOperation(promise: Promise) {
    CoroutineScope(Dispatchers.IO).launch {
        try {
            val result = performHeavyComputation()
            withContext(Dispatchers.Main) {
                promise.resolve(result)
            }
        } catch (e: Exception) {
            withContext(Dispatchers.Main) {
                promise.reject("ERROR", e)
            }
        }
    }
}
```

### Main Thread Execution

```java
import android.os.Handler;
import android.os.Looper;

@ReactMethod
public void updateUI(ReadableMap data) {
    // Ensure on main thread for UI updates
    new Handler(Looper.getMainLooper()).post(() -> {
        updateUserInterface(data);
    });
}
```

## Lifecycle Management

### Activity Lifecycle

```java
import com.facebook.react.bridge.LifecycleEventListener;

public class MyModule extends ReactContextBaseJavaModule 
    implements LifecycleEventListener {
    
    public MyModule(ReactApplicationContext context) {
        super(context);
        context.addLifecycleEventListener(this);
    }
    
    @Override
    public void onHostResume() {
        // App resumed
    }
    
    @Override
    public void onHostPause() {
        // App paused
    }
    
    @Override
    public void onHostDestroy() {
        // App destroyed
        cleanup();
    }
    
    @Override
    public void onCatalystInstanceDestroy() {
        // React instance destroyed
        reactContext.removeLifecycleEventListener(this);
    }
}
```

### Activity Event Listener

```java
import com.facebook.react.bridge.ActivityEventListener;
import android.app.Activity;
import android.content.Intent;

public class MyModule extends ReactContextBaseJavaModule 
    implements ActivityEventListener {
    
    public MyModule(ReactApplicationContext context) {
        super(context);
        context.addActivityEventListener(this);
    }
    
    @Override
    public void onActivityResult(Activity activity, int requestCode, 
                                 int resultCode, Intent data) {
        // Handle activity result
    }
    
    @Override
    public void onNewIntent(Intent intent) {
        // Handle new intent
    }
}
```

## Advanced Patterns

### Constants Export

```java
@Override
public Map<String, Object> getConstants() {
    Map<String, Object> constants = new HashMap<>();
    constants.put("API_URL", "https://api.example.com");
    constants.put("VERSION", "1.0.0");
    constants.put("MAX_RETRIES", 3);
    return constants;
}

// Access in JS: MyModule.API_URL
```

**Kotlin:**
```kotlin
override fun getConstants(): Map<String, Any> = mapOf(
    "API_URL" to "https://api.example.com",
    "VERSION" to "1.0.0",
    "MAX_RETRIES" to 3
)
```

### Native UI Components

```java
import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.annotations.ReactProp;

public class MyViewManager extends SimpleViewManager<MyCustomView> {
    
    @Override
    public String getName() {
        return "MyCustomView";
    }
    
    @Override
    protected MyCustomView createViewInstance(ThemedReactContext context) {
        return new MyCustomView(context);
    }
    
    @ReactProp(name = "color")
    public void setColor(MyCustomView view, String color) {
        view.setBackgroundColor(Color.parseColor(color));
    }
}
```

### Permissions Handling

```java
import android.Manifest;
import android.content.pm.PackageManager;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

@ReactMethod
public void requestCameraPermission(Promise promise) {
    Activity activity = getCurrentActivity();
    if (activity == null) {
        promise.reject("NO_ACTIVITY", "Activity not available");
        return;
    }
    
    if (ContextCompat.checkSelfPermission(activity, Manifest.permission.CAMERA)
            == PackageManager.PERMISSION_GRANTED) {
        promise.resolve(true);
    } else {
        ActivityCompat.requestPermissions(
            activity,
            new String[]{Manifest.permission.CAMERA},
            REQUEST_CAMERA_PERMISSION
        );
        // Handle result in onRequestPermissionsResult
    }
}
```

### File System Access

```java
import android.content.Context;
import java.io.File;
import java.io.FileWriter;

@ReactMethod
public void saveFile(String filename, String content, Promise promise) {
    try {
        File directory = reactContext.getFilesDir();
        File file = new File(directory, filename);
        
        FileWriter writer = new FileWriter(file);
        writer.write(content);
        writer.close();
        
        promise.resolve(file.getAbsolutePath());
    } catch (Exception e) {
        promise.reject("FILE_ERROR", "Failed to save file", e);
    }
}
```

### Broadcast Receivers

```java
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;

public class MyModule extends ReactContextBaseJavaModule {
    private BroadcastReceiver receiver;
    
    @ReactMethod
    public void startListening() {
        receiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                WritableMap data = Arguments.createMap();
                data.putString("action", intent.getAction());
                sendEvent("onBroadcast", data);
            }
        };
        
        IntentFilter filter = new IntentFilter();
        filter.addAction(Intent.ACTION_BATTERY_CHANGED);
        reactContext.registerReceiver(receiver, filter);
    }
    
    @ReactMethod
    public void stopListening() {
        if (receiver != null) {
            reactContext.unregisterReceiver(receiver);
            receiver = null;
        }
    }
}
```

## Error Handling

### Comprehensive Error Handling

```java
@ReactMethod
public void riskyOperation(Promise promise) {
    try {
        if (!isInitialized()) {
            promise.reject("NOT_INITIALIZED", "Module not initialized");
            return;
        }
        
        String result = performOperation();
        
        if (result == null) {
            promise.reject("NO_RESULT", "Operation returned no result");
            return;
        }
        
        promise.resolve(result);
        
    } catch (IllegalArgumentException e) {
        promise.reject("INVALID_ARGUMENT", e.getMessage(), e);
    } catch (SecurityException e) {
        promise.reject("PERMISSION_DENIED", e.getMessage(), e);
    } catch (Exception e) {
        promise.reject("UNKNOWN_ERROR", "Unexpected error occurred", e);
    }
}
```

## Testing

### Unit Testing Native Modules

```java
// MyModuleTest.java
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.Promise;

import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class MyModuleTest {
    
    @Mock
    private ReactApplicationContext reactContext;
    
    @Mock
    private Promise promise;
    
    private MyModule module;
    
    @Before
    public void setUp() {
        module = new MyModule(reactContext);
    }
    
    @Test
    public void testGetValue() {
        String result = module.getValue();
        assertEquals("expected value", result);
    }
    
    @Test
    public void testFetchDataSuccess() {
        module.fetchData(promise);
        verify(promise).resolve(any());
    }
    
    @Test
    public void testFetchDataFailure() {
        // Setup failure condition
        module.fetchData(promise);
        verify(promise).reject(eq("FETCH_ERROR"), anyString(), any());
    }
}
```

## Best Practices

1. **Use appropriate method types**: Synchronous sparingly, async for most operations
2. **Thread safety**: Always consider which thread methods run on
3. **Lifecycle management**: Clean up resources properly
4. **Error handling**: Provide detailed error codes and messages
5. **Type safety**: Use ReadableMap/WritableMap correctly
6. **Permissions**: Handle runtime permissions properly
7. **Testing**: Write unit tests for native code
8. **Performance**: Profile and optimize bridge calls

## Common Issues

**Module not found:**
- Check package is registered in MainApplication
- Verify getName() returns correct name
- Rebuild Android project

**Methods not callable:**
- Ensure @ReactMethod annotation is present
- Check method signature
- Verify module is exported

**Crashes:**
- Check thread safety
- Verify null checks
- Review error handling
- Check permissions

**Memory leaks:**
- Unregister receivers and listeners
- Clean up in onCatalystInstanceDestroy
- Remove lifecycle listeners
