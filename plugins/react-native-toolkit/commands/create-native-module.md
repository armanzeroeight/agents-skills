---
description: Create a new React Native native module with iOS and Android implementations
allowed-tools: Read, Write, Bash(mkdir:*,touch:*)
argument-hint: <module-name>
---

# Create Native Module

Generate boilerplate for a new React Native native module with iOS and Android implementations.

## Context

- Current directory: !`pwd`
- Existing modules: !`find . -name "*.java" -o -name "*.m" | grep -i module | head -5`

## Your Task

Create a new React Native native module with complete iOS and Android implementations.

### Arguments

- `$1`: Module name (e.g., "MyModule", "CameraHelper")

### Steps

1. **Validate module name**
   - Ensure name is provided
   - Check name follows PascalCase convention
   - Verify module doesn't already exist

2. **Create directory structure**
   ```
   <ModuleName>/
   ├── ios/
   │   ├── <ModuleName>.h
   │   └── <ModuleName>.m
   ├── android/
   │   └── src/main/java/com/<modulename>/
   │       ├── <ModuleName>Package.java
   │       └── <ModuleName>.java
   ├── js/
   │   └── Native<ModuleName>.ts
   └── package.json
   ```

3. **Generate iOS implementation**
   - Create header file (.h) with RCTBridgeModule protocol
   - Create implementation file (.m) with:
     - RCT_EXPORT_MODULE()
     - Example synchronous method
     - Example async method with Promise
     - Example event emitter setup

4. **Generate Android implementation**
   - Create module class extending ReactContextBaseJavaModule
   - Implement getName() method
   - Add example synchronous method
   - Add example async method with Promise
   - Create package class implementing ReactPackage

5. **Generate TypeScript interface**
   - Create type-safe interface for the module
   - Include example method signatures
   - Add JSDoc comments

6. **Create package.json**
   - Set up module metadata
   - Configure for auto-linking
   - Add codegenConfig for Turbo Modules (optional)

7. **Provide usage instructions**
   - How to link the module
   - How to use in JavaScript
   - How to test on iOS and Android
   - Next steps for implementation

### Output Format

```markdown
# Native Module Created: <ModuleName>

## Files Created
- iOS: ios/<ModuleName>.h, ios/<ModuleName>.m
- Android: android/src/main/java/com/<modulename>/<ModuleName>.java
- Android: android/src/main/java/com/<modulename>/<ModuleName>Package.java
- TypeScript: js/Native<ModuleName>.ts
- Config: package.json

## Usage

### JavaScript
\`\`\`javascript
import { NativeModules } from 'react-native';
const { <ModuleName> } = NativeModules;

// Use the module
const result = await <ModuleName>.exampleMethod();
\`\`\`

### iOS Setup
1. Run: cd ios && pod install
2. Rebuild: npx react-native run-ios

### Android Setup
1. Add to MainApplication.java:
   \`\`\`java
   new <ModuleName>Package()
   \`\`\`
2. Rebuild: npx react-native run-android

## Next Steps
1. Implement your custom methods in native code
2. Update TypeScript interface to match
3. Test on both platforms
4. Add error handling and validation
5. Write unit tests
```

## Examples

**Create a camera module:**
```
/create-native-module CameraHelper
```

**Create a storage module:**
```
/create-native-module SecureStorage
```

**Expected output:**
- Complete native module boilerplate
- iOS Objective-C implementation
- Android Java implementation
- TypeScript interface
- Setup instructions
- Usage examples
