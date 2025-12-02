# Turbo Modules

Guide to implementing Turbo Modules, the new React Native architecture for better performance and type safety.

## Contents
- What are Turbo Modules
- Setup and configuration
- Implementation guide
- Type safety with Codegen
- Migration from legacy bridge

## What are Turbo Modules

Turbo Modules are the new native module system in React Native (0.68+) that provides:

**Benefits:**
- **Better performance**: Lazy loading, reduced bridge overhead
- **Type safety**: TypeScript/Flow types generate native interfaces
- **Synchronous methods**: True synchronous calls without blocking
- **Smaller bundle**: Modules loaded on-demand

**Key differences from legacy bridge:**
- Modules are lazy-loaded (not all loaded at startup)
- Type-safe interface generated from TypeScript
- Better performance for synchronous operations
- Requires Codegen for type generation

## Setup and Configuration

### Enable New Architecture

**iOS (Podfile):**
```ruby
# Enable new architecture
ENV['RCT_NEW_ARCH_ENABLED'] = '1'

target 'YourApp' do
  # ...
end
```

**Android (gradle.properties):**
```properties
# Enable new architecture
newArchEnabled=true
```

### Install Dependencies

```bash
npm install react-native-codegen --save-dev
```

## Implementation Guide

### Step 1: Define TypeScript Spec

Create `NativeMyModule.ts`:
```typescript
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  // Synchronous methods
  getValue(): string;
  add(a: number, b: number): number;
  
  // Asynchronous methods (return Promise)
  fetchData(): Promise<{data: string; count: number}>;
  processData(input: string): Promise<string>;
  
  // With callbacks
  processWithCallback(
    input: string,
    callback: (error: string | null, result: string | null) => void
  ): void;
  
  // Constants
  readonly API_URL: string;
  readonly VERSION: string;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

### Step 2: Configure Codegen

**package.json:**
```json
{
  "name": "react-native-my-module",
  "version": "1.0.0",
  "codegenConfig": {
    "name": "MyModuleSpec",
    "type": "modules",
    "jsSrcsDir": "./src",
    "android": {
      "javaPackageName": "com.mymodule"
    }
  }
}
```

### Step 3: Implement iOS Module

**MyModule.h:**
```objc
#import <MyModuleSpec/MyModuleSpec.h>

NS_ASSUME_NONNULL_BEGIN

@interface MyModule : NSObject <NativeMyModuleSpec>
@end

NS_ASSUME_NONNULL_END
```

**MyModule.mm (note .mm extension):**
```objc
#import "MyModule.h"

@implementation MyModule

RCT_EXPORT_MODULE()

// Synchronous methods
- (NSString *)getValue {
  return @"value";
}

- (NSNumber *)add:(double)a b:(double)b {
  return @(a + b);
}

// Async with Promise
- (void)fetchData:(RCTPromiseResolveBlock)resolve
          reject:(RCTPromiseRejectBlock)reject {
  NSDictionary *result = @{
    @"data": @"value",
    @"count": @(42)
  };
  resolve(result);
}

- (void)processData:(NSString *)input
            resolve:(RCTPromiseResolveBlock)resolve
             reject:(RCTPromiseRejectBlock)reject {
  NSString *result = [self process:input];
  resolve(result);
}

// With callback
- (void)processWithCallback:(NSString *)input
                   callback:(RCTResponseSenderBlock)callback {
  NSString *result = [self process:input];
  callback(@[[NSNull null], result]);
}

// Constants
- (facebook::react::ModuleConstants<JS::NativeMyModule::Constants>)getConstants {
  return facebook::react::typedConstants<JS::NativeMyModule::Constants>({
    .API_URL = @"https://api.example.com",
    .VERSION = @"1.0.0"
  });
}

// Required for Turbo Modules
- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
    (const facebook::react::ObjCTurboModule::InitParams &)params {
  return std::make_shared<facebook::react::NativeMyModuleSpecJSI>(params);
}

@end
```

### Step 4: Implement Android Module

**MyModule.java:**
```java
package com.mymodule;

import androidx.annotation.NonNull;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;

public class MyModule extends NativeMyModuleSpec {
    public static final String NAME = "MyModule";

    public MyModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    @NonNull
    public String getName() {
        return NAME;
    }

    // Synchronous methods
    @Override
    public String getValue() {
        return "value";
    }

    @Override
    public double add(double a, double b) {
        return a + b;
    }

    // Async with Promise
    @Override
    public void fetchData(Promise promise) {
        WritableMap result = Arguments.createMap();
        result.putString("data", "value");
        result.putInt("count", 42);
        promise.resolve(result);
    }

    @Override
    public void processData(String input, Promise promise) {
        String result = process(input);
        promise.resolve(result);
    }

    // With callback
    @Override
    public void processWithCallback(String input, Callback callback) {
        String result = process(input);
        callback.invoke(null, result);
    }

    // Constants
    @Override
    protected Map<String, Object> getTypedExportedConstants() {
        Map<String, Object> constants = new HashMap<>();
        constants.put("API_URL", "https://api.example.com");
        constants.put("VERSION", "1.0.0");
        return constants;
    }
}
```

**MyModulePackage.java:**
```java
package com.mymodule;

import com.facebook.react.TurboReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.model.ReactModuleInfo;
import com.facebook.react.module.model.ReactModuleInfoProvider;

import java.util.HashMap;
import java.util.Map;

public class MyModulePackage extends TurboReactPackage {
    @Override
    public NativeModule getModule(String name, ReactApplicationContext reactContext) {
        if (name.equals(MyModule.NAME)) {
            return new MyModule(reactContext);
        }
        return null;
    }

    @Override
    public ReactModuleInfoProvider getReactModuleInfoProvider() {
        return () -> {
            Map<String, ReactModuleInfo> moduleInfos = new HashMap<>();
            moduleInfos.put(
                MyModule.NAME,
                new ReactModuleInfo(
                    MyModule.NAME,
                    MyModule.NAME,
                    false, // canOverrideExistingModule
                    false, // needsEagerInit
                    true,  // isCxxModule
                    true   // isTurboModule
                )
            );
            return moduleInfos;
        };
    }
}
```

## Type Safety with Codegen

### Supported Types

**Primitives:**
```typescript
export interface Spec extends TurboModule {
  getString(): string;
  getNumber(): number;
  getBoolean(): boolean;
  getNull(): null;
}
```

**Objects:**
```typescript
export interface Spec extends TurboModule {
  getUser(): {
    name: string;
    age: number;
    active: boolean;
  };
}
```

**Arrays:**
```typescript
export interface Spec extends TurboModule {
  getItems(): Array<string>;
  getNumbers(): Array<number>;
  getObjects(): Array<{id: string; value: number}>;
}
```

**Promises:**
```typescript
export interface Spec extends TurboModule {
  fetchData(): Promise<{data: string}>;
  processAsync(input: string): Promise<string>;
}
```

**Callbacks:**
```typescript
export interface Spec extends TurboModule {
  processWithCallback(
    input: string,
    callback: (error: string | null, result: string | null) => void
  ): void;
}
```

**Optional parameters:**
```typescript
export interface Spec extends TurboModule {
  doSomething(required: string, optional?: number): void;
}
```

### Type Aliases

```typescript
type User = {
  id: string;
  name: string;
  email: string;
};

type Result = {
  success: boolean;
  data: User | null;
  error: string | null;
};

export interface Spec extends TurboModule {
  getUser(id: string): Promise<User>;
  processUser(user: User): Promise<Result>;
}
```

### Enums

```typescript
export enum Status {
  IDLE = 'idle',
  LOADING = 'loading',
  SUCCESS = 'success',
  ERROR = 'error',
}

export interface Spec extends TurboModule {
  getStatus(): Status;
  setStatus(status: Status): void;
}
```

## Advanced Patterns

### Event Emitters

```typescript
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  addListener(eventName: string): void;
  removeListeners(count: number): void;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

**iOS implementation:**
```objc
#import <React/RCTEventEmitter.h>

@interface MyModule : RCTEventEmitter <NativeMyModuleSpec>
@end

@implementation MyModule

- (NSArray<NSString *> *)supportedEvents {
  return @[@"onUpdate"];
}

- (void)addListener:(NSString *)eventName {
  // Setup
}

- (void)removeListeners:(double)count {
  // Cleanup
}

- (void)sendUpdate:(NSDictionary *)data {
  [self sendEventWithName:@"onUpdate" body:data];
}

@end
```

### Native UI Components

```typescript
import type { ViewProps } from 'react-native';
import type { HostComponent } from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

export interface NativeProps extends ViewProps {
  color?: string;
  onPress?: () => void;
}

export default codegenNativeComponent<NativeProps>('MyCustomView');
```

## Migration from Legacy Bridge

### Step 1: Create TypeScript Spec

Convert existing module to TypeScript spec:

**Before (legacy):**
```javascript
import { NativeModules } from 'react-native';
const { MyModule } = NativeModules;
export default MyModule;
```

**After (Turbo Module):**
```typescript
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  getValue(): string;
  fetchData(): Promise<{data: string}>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

### Step 2: Update Native Implementation

**iOS - Change to .mm file:**
```objc
// Add Turbo Module support
- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
    (const facebook::react::ObjCTurboModule::InitParams &)params {
  return std::make_shared<facebook::react::NativeMyModuleSpecJSI>(params);
}
```

**Android - Extend generated spec:**
```java
// Before
public class MyModule extends ReactContextBaseJavaModule

// After
public class MyModule extends NativeMyModuleSpec
```

### Step 3: Update Package

**Android:**
```java
// Before
public class MyModulePackage implements ReactPackage

// After
public class MyModulePackage extends TurboReactPackage
```

### Step 4: Run Codegen

```bash
# iOS
cd ios && pod install

# Android
cd android && ./gradlew generateCodegenArtifactsFromSchema
```

## Testing

### Unit Testing Turbo Modules

**TypeScript tests:**
```typescript
import MyModule from './NativeMyModule';

describe('MyModule', () => {
  it('should return value', () => {
    const result = MyModule.getValue();
    expect(result).toBe('expected value');
  });

  it('should fetch data', async () => {
    const result = await MyModule.fetchData();
    expect(result.data).toBeDefined();
  });
});
```

## Best Practices

1. **Type everything**: Use TypeScript for full type safety
2. **Lazy loading**: Turbo Modules load on-demand
3. **Synchronous sparingly**: Use for simple getters only
4. **Error handling**: Provide detailed error types
5. **Documentation**: Document all methods and types
6. **Testing**: Write tests for both JS and native code
7. **Migration**: Migrate gradually, test thoroughly

## Common Issues

**Codegen not running:**
- Check codegenConfig in package.json
- Verify TypeScript spec syntax
- Run pod install (iOS) or gradle build (Android)

**Type mismatches:**
- Ensure TypeScript types match native implementation
- Check Codegen output for errors
- Verify all methods are implemented

**Module not found:**
- Check TurboModuleRegistry.getEnforcing name
- Verify native module name matches
- Rebuild native code

**Performance issues:**
- Profile bridge calls
- Minimize data serialization
- Use synchronous methods appropriately

## Resources

- [React Native New Architecture](https://reactnative.dev/docs/the-new-architecture/landing-page)
- [Turbo Modules Guide](https://reactnative.dev/docs/the-new-architecture/pillars-turbomodules)
- [Codegen Documentation](https://reactnative.dev/docs/the-new-architecture/pillars-codegen)
