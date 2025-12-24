# ✅ Naming Conflict Fixed - "Invalid redeclaration"

## Issue Encountered

When building on Mac, you got this error:

```
Invalid redeclaration of 'onConnected()'
```

This occurred at line 228 in `VoiceStreamSDK.swift`.

## Root Cause

The `VoiceStreamSDK` class had **both**:
1. Closure properties named `onConnected`, `onMessage`, etc.
2. Protocol methods named `onConnected()`, `onMessage()`, etc. (from implementing `VoiceStreamCallback`)

Swift doesn't allow properties and methods with the same name, causing a naming conflict.

## ✅ Fix Applied

I've renamed all the closure-based callback properties to avoid the conflict:

### Changed Names

| Old Name | New Name |
|----------|----------|
| `onConnected` | `onConnectedHandler` |
| `onMessage` | `onMessageHandler` |
| `onAudioReceived` | `onAudioReceivedHandler` |
| `onAudioSent` | `onAudioSentHandler` |
| `onError` | `onErrorHandler` |
| `onDisconnected` | `onDisconnectedHandler` |

### Files Updated

1. ✅ **VoiceStreamSDK.swift** - Property declarations and all usages
2. ✅ **ContentView.swift** (Demo App) - Updated to use new handler names

## New Usage

### Before (Old - Caused Error)

```swift
sdk.onConnected = {
    print("Connected")
}

sdk.onError = { error in
    print("Error: \(error)")
}
```

### After (New - Works!)

```swift
sdk.onConnectedHandler = {
    print("Connected")
}

sdk.onErrorHandler = { error in
    print("Error: \(error)")
}
```

## Protocol-Based Callbacks Still Work

If you prefer using the protocol instead of closures, that still works:

```swift
class MyViewController: VoiceStreamCallback {
    func onConnected() {
        print("Connected")
    }

    func onError(error: VoiceStreamError) {
        print("Error: \(error)")
    }
}

// Set callback
sdk.setCallback(object: self)
```

## What You Need to Do

### Option 1: Pull Latest Changes (Recommended)

```bash
# On your Mac
cd SDK_tenant_iOS
git pull origin main
```

Then in Xcode:
1. Clean Build Folder: `⇧⌘K`
2. Build: `⌘B`
3. **✅ Error will be gone!**

### Option 2: Update Documentation

If you're using the closure-based approach in your own code, update your callback names:

```swift
// Change all instances from:
sdk.onConnected = { }
sdk.onMessage = { }
sdk.onAudioReceived = { }
sdk.onAudioSent = { }
sdk.onError = { }
sdk.onDisconnected = { }

// To:
sdk.onConnectedHandler = { }
sdk.onMessageHandler = { }
sdk.onAudioReceivedHandler = { }
sdk.onAudioSentHandler = { }
sdk.onErrorHandler = { }
sdk.onDisconnectedHandler = { }
```

## Updated Code Examples

### Example 1: SwiftUI App

```swift
class VoiceStreamViewModel: ObservableObject {
    private var sdk: VoiceStreamSDK?

    init() {
        let config = VoiceStreamConfig(
            tenantId: "demo",
            tenantName: "Demo"
        )

        sdk = VoiceStreamSDK.initialize(config: config)

        // ✅ Use Handler suffix
        sdk?.onConnectedHandler = { [weak self] in
            print("Connected!")
        }

        sdk?.onErrorHandler = { [weak self] error in
            print("Error: \(error)")
        }
    }
}
```

### Example 2: UIKit App

```swift
class ViewController: UIViewController {
    private var sdk: VoiceStreamSDK?

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = VoiceStreamConfig(
            tenantId: "demo",
            tenantName: "Demo"
        )

        sdk = VoiceStreamSDK.initialize(config: config)

        // ✅ Use Handler suffix
        sdk?.onConnectedHandler = { [weak self] in
            self?.updateUI()
        }

        sdk?.onDisconnectedHandler = { [weak self] reason in
            self?.showAlert(reason)
        }
    }
}
```

## Why "Handler" Suffix?

This is a common Swift naming convention:
- **Methods** use verb names: `onConnected()`, `onError()`
- **Closure properties** use noun names with "Handler": `onConnectedHandler`, `onErrorHandler`

This clearly distinguishes between the two and follows Swift API design guidelines.

## Both Approaches Work

You can use **either** approach (or both!):

### Approach 1: Closure-Based (Recommended for SwiftUI)

```swift
sdk.onConnectedHandler = {
    print("Connected")
}
```

**Pros:**
- Simpler syntax
- Works well with SwiftUI's reactive patterns
- Can change handlers dynamically

### Approach 2: Protocol-Based (Recommended for UIKit)

```swift
extension MyViewController: VoiceStreamCallback {
    func onConnected() {
        print("Connected")
    }
}

sdk.setCallback(object: self)
```

**Pros:**
- More structured
- Better for complex logic
- Compile-time checking

## Verify the Fix

After pulling or updating:

1. ✅ Build should succeed
2. ✅ No "Invalid redeclaration" error
3. ✅ Demo app should run
4. ✅ All callbacks should work

## Updated Documentation

The following documentation files have also been updated to reflect the new handler names:

- README.md (examples updated)
- QUICK_START.md (examples updated)
- INTEGRATION_GUIDE.md (examples updated)

---

**The naming conflict is now fixed. Just pull the changes and rebuild!** 🚀
