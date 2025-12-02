# iOS Bridge Implementation

Comprehensive guide to implementing React Native native modules for iOS using Objective-C and Swift.

## Contents
- Objective-C implementation
- Swift implementation
- Threading and performance
- Memory management
- Advanced patterns

## Objective-C Implementation

### Basic Module Setup

**Header file (MyModule.h):**
```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface MyModule : RCTEventEmitter <RCTBridgeModule>
@end
```

**Implementation file (MyModule.m):**
```objc
#import "MyModule.h"

@implementation MyModule

// Module name (used in JS: NativeModules.MyModule)
RCT_EXPORT_MODULE();

// Or specify custom name
// RCT_EXPORT_MODULE(CustomName);

@end
```

### Method Export

**Synchronous methods:**
```objc
// Returns immediately (blocks JS thread)
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getValue)
{
  return @"value";
}

RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(add:(NSInteger)a b:(NSInteger)b)
{
  return @(a + b);
}
```

**Asynchronous with Promise:**
```objc
RCT_EXPORT_METHOD(fetchData:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  // Perform async operation
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSError *error = nil;
    NSDictionary *result = [self loadData:&error];
    
    dispatch_async(dispatch_get_main_queue(), ^{
      if (error) {
        reject(@"FETCH_ERROR", @"Failed to fetch data", error);
      } else {
        resolve(result);
      }
    });
  });
}
```

**Asynchronous with Callback:**
```objc
RCT_EXPORT_METHOD(processData:(NSString *)input
                  callback:(RCTResponseSenderBlock)callback)
{
  // Process data
  NSString *result = [self process:input];
  
  // Callback format: [error, result]
  if (result) {
    callback(@[[NSNull null], result]);
  } else {
    callback(@[@"Processing failed", [NSNull null]]);
  }
}
```

### Event Emitters

```objc
@interface MyModule : RCTEventEmitter <RCTBridgeModule>
@end

@implementation MyModule

RCT_EXPORT_MODULE();

// Declare supported events
- (NSArray<NSString *> *)supportedEvents
{
  return @[@"onUpdate", @"onError", @"onComplete"];
}

// Send event to JavaScript
- (void)sendUpdate:(NSDictionary *)data
{
  [self sendEventWithName:@"onUpdate" body:data];
}

// Start observing (called when first listener added)
- (void)startObserving
{
  // Set up observers, timers, etc.
}

// Stop observing (called when last listener removed)
- (void)stopObserving
{
  // Clean up observers, timers, etc.
}

@end
```

### Data Type Conversion

**JavaScript to Objective-C:**
```objc
RCT_EXPORT_METHOD(handleData:(NSDictionary *)data)
{
  // String
  NSString *name = [RCTConvert NSString:data[@"name"]];
  
  // Number
  NSNumber *age = [RCTConvert NSNumber:data[@"age"]];
  NSInteger ageInt = [age integerValue];
  
  // Boolean
  BOOL isActive = [RCTConvert BOOL:data[@"active"]];
  
  // Array
  NSArray *items = [RCTConvert NSArray:data[@"items"]];
  
  // Dictionary
  NSDictionary *settings = [RCTConvert NSDictionary:data[@"settings"]];
  
  // Date
  NSDate *date = [RCTConvert NSDate:data[@"timestamp"]];
  
  // URL
  NSURL *url = [RCTConvert NSURL:data[@"url"]];
}
```

**Objective-C to JavaScript:**
```objc
RCT_EXPORT_METHOD(getData:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  NSDictionary *result = @{
    @"string": @"value",
    @"number": @(42),
    @"float": @(3.14),
    @"boolean": @(YES),
    @"array": @[@"a", @"b", @"c"],
    @"dictionary": @{
      @"nested": @"value"
    },
    @"null": [NSNull null]
  };
  
  resolve(result);
}
```

## Swift Implementation

### Setup Bridging Header

**Create MyModule-Bridging-Header.h:**
```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>
```

**Configure in Xcode:**
- Build Settings → Objective-C Bridging Header
- Set to: `MyModule-Bridging-Header.h`

### Basic Module

```swift
import Foundation

@objc(MyModule)
class MyModule: NSObject {
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return false  // true if needs main thread initialization
  }
  
  @objc
  func constantsToExport() -> [AnyHashable : Any]! {
    return [
      "API_URL": "https://api.example.com",
      "VERSION": "1.0.0"
    ]
  }
}
```

### Method Export

```swift
@objc(MyModule)
class MyModule: NSObject {
  
  // Synchronous method
  @objc
  func getValue() -> String {
    return "value"
  }
  
  // Async with Promise
  @objc
  func fetchData(_ resolve: @escaping RCTPromiseResolveBlock,
                 rejecter reject: @escaping RCTPromiseRejectBlock) {
    DispatchQueue.global(qos: .background).async {
      do {
        let result = try self.loadData()
        resolve(result)
      } catch {
        reject("FETCH_ERROR", "Failed to fetch data", error)
      }
    }
  }
  
  // Async with callback
  @objc
  func processData(_ input: String,
                   callback: @escaping RCTResponseSenderBlock) {
    let result = process(input)
    callback([NSNull(), result])
  }
}
```

**Export to React Native (create MyModule.m):**
```objc
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(MyModule, NSObject)

RCT_EXTERN_METHOD(getValue)

RCT_EXTERN_METHOD(fetchData:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

RCT_EXTERN_METHOD(processData:(NSString *)input
                  callback:(RCTResponseSenderBlock)callback)

@end
```

### Event Emitters in Swift

```swift
@objc(MyModule)
class MyModule: RCTEventEmitter {
  
  override func supportedEvents() -> [String]! {
    return ["onUpdate", "onError"]
  }
  
  override static func requiresMainQueueSetup() -> Bool {
    return false
  }
  
  func sendUpdate(_ data: [String: Any]) {
    sendEvent(withName: "onUpdate", body: data)
  }
  
  override func startObserving() {
    // Setup observers
  }
  
  override func stopObserving() {
    // Cleanup
  }
}
```

## Threading and Performance

### Background Thread Execution

```objc
RCT_EXPORT_METHOD(heavyOperation:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  // Execute on background thread
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // Heavy computation
    NSString *result = [self performHeavyComputation];
    
    // Return to main thread for resolve
    dispatch_async(dispatch_get_main_queue(), ^{
      resolve(result);
    });
  });
}
```

### Main Thread Execution

```objc
RCT_EXPORT_METHOD(updateUI:(NSDictionary *)data)
{
  // Ensure on main thread for UI updates
  dispatch_async(dispatch_get_main_queue(), ^{
    // Update UI
    [self updateUserInterface:data];
  });
}
```

### Method Queue

```objc
@implementation MyModule

// Specify custom queue for all methods
- (dispatch_queue_t)methodQueue
{
  return dispatch_queue_create("com.myapp.mymodule", DISPATCH_QUEUE_SERIAL);
}

@end
```

## Memory Management

### Weak References

```objc
@implementation MyModule

RCT_EXPORT_METHOD(startMonitoring)
{
  __weak typeof(self) weakSelf = self;
  
  self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                                                repeats:YES
                                                  block:^(NSTimer *timer) {
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (strongSelf) {
      [strongSelf sendUpdate:@{@"timestamp": @([[NSDate date] timeIntervalSince1970])}];
    }
  }];
}

RCT_EXPORT_METHOD(stopMonitoring)
{
  [self.timer invalidate];
  self.timer = nil;
}

@end
```

### Cleanup

```objc
@implementation MyModule

- (void)dealloc
{
  // Clean up resources
  [self.timer invalidate];
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}

- (void)invalidate
{
  // Called when bridge is invalidated
  [self cleanup];
}

@end
```

## Advanced Patterns

### Constants Export

```objc
- (NSDictionary *)constantsToExport
{
  return @{
    @"API_URL": @"https://api.example.com",
    @"VERSION": @"1.0.0",
    @"MAX_RETRIES": @(3)
  };
}

// Access in JS: MyModule.API_URL
```

### Native UI Components

```objc
#import <React/RCTViewManager.h>

@interface MyViewManager : RCTViewManager
@end

@implementation MyViewManager

RCT_EXPORT_MODULE()

- (UIView *)view
{
  return [[MyCustomView alloc] init];
}

// Export properties
RCT_EXPORT_VIEW_PROPERTY(color, UIColor)
RCT_EXPORT_VIEW_PROPERTY(onPress, RCTBubblingEventBlock)

@end
```

### Notification Observers

```objc
@implementation MyModule

- (instancetype)init
{
  if (self = [super init]) {
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(handleNotification:)
                                                 name:UIApplicationDidBecomeActiveNotification
                                               object:nil];
  }
  return self;
}

- (void)handleNotification:(NSNotification *)notification
{
  [self sendEventWithName:@"onAppActive" body:nil];
}

- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}

@end
```

### File System Access

```objc
RCT_EXPORT_METHOD(saveFile:(NSString *)filename
                  content:(NSString *)content
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
  NSString *documentsDirectory = [paths firstObject];
  NSString *filePath = [documentsDirectory stringByAppendingPathComponent:filename];
  
  NSError *error = nil;
  BOOL success = [content writeToFile:filePath
                           atomically:YES
                             encoding:NSUTF8StringEncoding
                                error:&error];
  
  if (success) {
    resolve(filePath);
  } else {
    reject(@"FILE_ERROR", @"Failed to save file", error);
  }
}
```

## Error Handling

### Comprehensive Error Handling

```objc
RCT_EXPORT_METHOD(riskyOperation:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  @try {
    NSError *error = nil;
    id result = [self performOperation:&error];
    
    if (error) {
      // Structured error with code, message, and details
      reject(@"OPERATION_FAILED",
             error.localizedDescription,
             error);
    } else if (!result) {
      // No error but no result
      reject(@"NO_RESULT",
             @"Operation completed but returned no result",
             nil);
    } else {
      resolve(result);
    }
  }
  @catch (NSException *exception) {
    // Handle exceptions
    reject(@"EXCEPTION",
           exception.reason,
           [NSError errorWithDomain:@"MyModule"
                               code:-1
                           userInfo:@{NSLocalizedDescriptionKey: exception.reason}]);
  }
}
```

## Testing

### Unit Testing Native Modules

```objc
// MyModuleTests.m
#import <XCTest/XCTest.h>
#import "MyModule.h"

@interface MyModuleTests : XCTestCase
@property (nonatomic, strong) MyModule *module;
@end

@implementation MyModuleTests

- (void)setUp
{
  [super setUp];
  self.module = [[MyModule alloc] init];
}

- (void)testGetValue
{
  NSString *result = [self.module getValue];
  XCTAssertEqualObjects(result, @"expected value");
}

- (void)testAsyncOperation
{
  XCTestExpectation *expectation = [self expectationWithDescription:@"Async operation"];
  
  [self.module fetchData:^(id result) {
    XCTAssertNotNil(result);
    [expectation fulfill];
  } rejecter:^(NSString *code, NSString *message, NSError *error) {
    XCTFail(@"Should not reject");
  }];
  
  [self waitForExpectationsWithTimeout:5.0 handler:nil];
}

@end
```

## Best Practices

1. **Use appropriate method types**: Synchronous for simple getters, async for everything else
2. **Thread safety**: Always consider which thread methods run on
3. **Memory management**: Use weak references in blocks, clean up resources
4. **Error handling**: Provide detailed error information
5. **Type safety**: Use RCTConvert for type conversions
6. **Documentation**: Document all exported methods
7. **Testing**: Write unit tests for native code
8. **Performance**: Profile and optimize bridge calls

## Common Issues

**Module not found:**
- Check RCT_EXPORT_MODULE() is present
- Verify module name matches
- Rebuild iOS project

**Methods not callable:**
- Ensure RCT_EXPORT_METHOD is used
- Check method signature
- Verify bridging header (Swift)

**Crashes:**
- Check thread safety
- Verify memory management
- Add null checks
- Review error handling

**Memory leaks:**
- Use weak references in blocks
- Clean up observers and timers
- Implement dealloc properly
