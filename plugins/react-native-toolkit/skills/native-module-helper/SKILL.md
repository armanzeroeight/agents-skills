---
name: native-module-helper
description: Create custom React Native native modules for iOS and Android. Use when integrating native SDKs, optimizing performance-critical code, or accessing platform-specific APIs. Trigger words include "native module", "bridge", "native code", "iOS bridge", "Android bridge", "Turbo Module".
---

# Native Module Helper

Build custom native modules to bridge JavaScript and native code in React Native.

## Quick Start

Native modules expose native functionality to JavaScript. Choose based on React Native version:
- **Legacy Bridge**: RN < 0.68 (stable, widely supported)
- **Turbo Modules**: RN >= 0.68 (better performance, type-safe)

## Instructions

### Step 1: Plan Module Interface

**Design JavaScript API:**
```javascript
// What you want to call from JS
import { NativeModules } from 'react-native';
const { MyModule } = NativeModules;

// Synchronous
const result = MyModule.getValue();

// Asynchronous (Promise)
const data = await MyModule.fetchData();

// With callback
MyModule.processData(input, (error, result) => {
  if (error) console.error(error);
  else console.log(result);
});

// Event emitter
MyModule.addListener('onUpdate', (event) => {
  console.log(event);
});
```

**Keep bridge calls minimal:**
- Batch operations when possible
- Avoid frequent small calls
- Use events for continuous updates

### Step 2: Create Module Structure

**File structure:**
```
MyModule/
├── ios/
│   ├── MyModule.h
│   ├── MyModule.m (or .swift)
│   └── MyModule-Bridging-Header.h (if Swift)
├── android/
│   └── src/main/java/com/mymodule/
│       ├── MyModulePackage.java
│       └── MyModule.java (or .kt)
├── js/
│   └── NativeMyModule.ts
└── package.json
```

### Step 3: Implement iOS Module

**Objective-C (.h file):**
```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface MyModule : RCTEventEmitter <RCTBridgeModule>
@end
```

**Objective-C (.m file):**
```objc
#import "MyModule.h"

@implementation MyModule

RCT_EXPORT_MODULE();

// Synchronous method
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getValue)
{
  return @"value";
}

// Async with Promise
RCT_EXPORT_METHOD(fetchData:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  // Perform operation
  if (success) {
    resolve(@{@"data": result});
  } else {
    reject(@"ERROR_CODE", @"Error message", error);
  }
}

// Async with callback
RCT_EXPORT_METHOD(processData:(NSString *)input
                  callback:(RCTResponseSenderBlock)callback)
{
  // Process data
  callback(@[[NSNull null], result]); // [error, result]
}

// Event emitter
- (NSArray<NSString *> *)supportedEvents
{
  return @[@"onUpdate"];
}

- (void)sendUpdate:(NSDictionary *)data
{
  [self sendEventWithName:@"onUpdate" body:data];
}

@end
```

### Step 4: Implement Android Module

**Java module:**
```java
package com.mymodule;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.modules.core.DeviceEventManagerModule;

public class MyModule extends ReactContextBaseJavaModule {
    private ReactApplicationContext reactContext;

    public MyModule(ReactApplicationContext context) {
        super(context);
        this.reactContext = context;
    }

    @Override
    public String getName() {
        return "MyModule";
    }

    // Synchronous method
    @ReactMethod(isBlockingSynchronousMethod = true)
    public String getValue() {
        return "value";
    }

    // Async with Promise
    @ReactMethod
    public void fetchData(Promise promise) {
        try {
            WritableMap result = Arguments.createMap();
            result.putString("data", "value");
            promise.resolve(result);
        } catch (Exception e) {
            promise.reject("ERROR_CODE", "Error message", e);
        }
    }

    // Async with callback
    @ReactMethod
    public void processData(String input, Callback callback) {
        try {
            String result = process(input);
            callback.invoke(null, result); // error, result
        } catch (Exception e) {
            callback.invoke(e.getMessage(), null);
        }
    }

    // Event emitter
    private void sendEvent(String eventName, WritableMap params) {
        reactContext
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
            .emit(eventName, params);
    }
}
```

**Package registration:**
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

### Step 5: Link Module

**Auto-linking (RN >= 0.60):**

Create `package.json` in module root:
```json
{
  "name": "react-native-my-module",
  "version": "1.0.0",
  "main": "js/index.js",
  "react-native": "js/index.js"
}
```

**Manual linking (if needed):**

iOS: Add to Podfile
```ruby
pod 'MyModule', :path => '../node_modules/react-native-my-module'
```

Android: Add to `settings.gradle`
```gradle
include ':react-native-my-module'
project(':react-native-my-module').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-my-module/android')
```

### Step 6: Create TypeScript Interface

```typescript
// js/NativeMyModule.ts
import { NativeModules, NativeEventEmitter } from 'react-native';

interface MyModuleInterface {
  getValue(): string;
  fetchData(): Promise<{ data: string }>;
  processData(input: string, callback: (error: string | null, result: string | null) => void): void;
  addListener(eventName: string, listener: (event: any) => void): void;
  removeListeners(count: number): void;
}

const { MyModule } = NativeModules;
const eventEmitter = new NativeEventEmitter(MyModule);

export default MyModule as MyModuleInterface;
export { eventEmitter };
```

## Common Patterns

### Threading

**iOS (run on background thread):**
```objc
RCT_EXPORT_METHOD(heavyTask:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // Heavy operation
    NSString *result = [self performHeavyOperation];
    
    dispatch_async(dispatch_get_main_queue(), ^{
      resolve(result);
    });
  });
}
```

**Android (run on background thread):**
```java
@ReactMethod
public void heavyTask(Promise promise) {
    new Thread(() -> {
        try {
            String result = performHeavyOperation();
            promise.resolve(result);
        } catch (Exception e) {
            promise.reject("ERROR", e);
        }
    }).start();
}
```

### Data Type Conversion

**iOS:**
```objc
// JS -> Native
NSString *string = [RCTConvert NSString:value];
NSNumber *number = [RCTConvert NSNumber:value];
NSArray *array = [RCTConvert NSArray:value];
NSDictionary *dict = [RCTConvert NSDictionary:value];

// Native -> JS
return @{
  @"string": @"value",
  @"number": @(42),
  @"array": @[@"a", @"b"],
  @"dict": @{@"key": @"value"}
};
```

**Android:**
```java
// JS -> Native (automatic)
String string = input;
int number = input;
ReadableArray array = input;
ReadableMap map = input;

// Native -> JS
WritableMap result = Arguments.createMap();
result.putString("string", "value");
result.putInt("number", 42);

WritableArray array = Arguments.createArray();
array.pushString("a");
array.pushString("b");
result.putArray("array", array);
```

### Error Handling

**iOS:**
```objc
RCT_EXPORT_METHOD(riskyOperation:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  NSError *error = nil;
  id result = [self performOperation:&error];
  
  if (error) {
    reject(@"OPERATION_FAILED", error.localizedDescription, error);
  } else {
    resolve(result);
  }
}
```

**Android:**
```java
@ReactMethod
public void riskyOperation(Promise promise) {
    try {
        Object result = performOperation();
        promise.resolve(result);
    } catch (Exception e) {
        promise.reject("OPERATION_FAILED", e.getMessage(), e);
    }
}
```

## Advanced

For detailed platform-specific guides:
- [iOS Bridge](reference/ios-bridge.md) - Objective-C and Swift implementation
- [Android Bridge](reference/android-bridge.md) - Java and Kotlin implementation
- [Turbo Modules](reference/turbo-modules.md) - New architecture modules

## Troubleshooting

**Module not found:**
- Verify package.json configuration
- Run `pod install` (iOS) or rebuild (Android)
- Check module name matches in native code

**Methods not available:**
- Ensure RCT_EXPORT_METHOD is used
- Check method signature matches
- Rebuild native code

**Crashes on method call:**
- Check thread safety
- Verify data type conversions
- Add null checks
- Review error handling

**Events not received:**
- Verify supportedEvents (iOS)
- Check event emitter setup
- Ensure listeners are added before events fire

## Best Practices

1. **Minimize bridge calls**: Batch operations, use events for updates
2. **Type safety**: Use TypeScript interfaces
3. **Error handling**: Always handle errors gracefully
4. **Threading**: Move heavy operations off main thread
5. **Memory management**: Clean up resources, remove listeners
6. **Testing**: Test on both iOS and Android
7. **Documentation**: Document API clearly
8. **Versioning**: Use semantic versioning
