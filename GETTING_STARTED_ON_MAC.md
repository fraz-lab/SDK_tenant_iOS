# Getting Started on Mac - VoiceStreamSDK iOS

Welcome! This guide will help you open and test the iOS SDK on your Mac.

## What You Have

A complete iOS SDK for real-time voice streaming, including:

✅ **VoiceStreamSDK** - Swift Package with all core features
✅ **Demo App** - Complete reference implementation
✅ **Documentation** - Comprehensive guides and API reference

## Quick Start on Mac

### Step 1: Open the Project

You have two options:

#### Option A: Open SDK Package (Recommended for development)

```bash
cd SDK_tenant_iOS/VoiceStreamSDK
open Package.swift
```

This opens the SDK in Xcode for development and testing.

#### Option B: Open Demo App (Recommended for testing)

```bash
cd SDK_tenant_iOS/DemoApp
open DemoApp.xcodeproj
```

This opens the demo application.

### Step 2: Build the SDK

If you opened the SDK package:

1. Xcode will automatically fetch dependencies (Starscream)
2. Wait for package resolution to complete
3. Press `⌘B` to build
4. Check for any build errors

### Step 3: Run the Demo App

If you opened the demo app:

1. Select a target device (iPhone or iPad simulator)
2. Press `⌘R` to build and run
3. Grant microphone permission when prompted
4. Test the connection:
   - Tap **"Connect"** button
   - Wait for "Connected" status
   - Tap **"Start Streaming"** button
   - Speak into your microphone
   - Check metrics and event log

## Project Structure

```
SDK_tenant_iOS/
│
├── VoiceStreamSDK/              ← SDK Package (Swift Package Manager)
│   ├── Package.swift            ← Open this for SDK development
│   ├── Sources/
│   │   └── VoiceStreamSDK/      ← All SDK source files
│   └── Tests/
│
├── DemoApp/
│   ├── DemoApp.xcodeproj        ← Open this to test the app
│   └── DemoApp/
│       └── ContentView.swift    ← Main demo UI
│
├── README.md                    ← Start here for documentation
├── QUICK_START.md              ← 5-minute integration guide
├── INTEGRATION_GUIDE.md        ← Detailed integration steps
├── PROJECT_SUMMARY.md          ← Complete technical overview
└── GETTING_STARTED_ON_MAC.md   ← This file
```

## Common Tasks

### Build SDK from Terminal

```bash
cd SDK_tenant_iOS/VoiceStreamSDK
swift build
```

### Run Tests

```bash
cd SDK_tenant_iOS/VoiceStreamSDK
swift test
```

### Build Demo App from Terminal

```bash
cd SDK_tenant_iOS/DemoApp
xcodebuild -project DemoApp.xcodeproj -scheme DemoApp -sdk iphonesimulator
```

### Clean Build

In Xcode:
- Press `⇧⌘K` (Shift + Command + K)
- Or: Product → Clean Build Folder

## Linking SDK to Demo App

The demo app needs to be linked to the SDK. Here's how:

### Option 1: Local Package Reference (Recommended)

1. Open `DemoApp.xcodeproj`
2. Select the project in the navigator
3. Go to **General** tab
4. Under **Frameworks, Libraries, and Embedded Content**, click `+`
5. Click **Add Other...** → **Add Package Dependency...**
6. Click **Add Local...** and select the `VoiceStreamSDK` folder
7. Select **VoiceStreamSDK** library and add

### Option 2: Direct Framework Link

If the local package doesn't work:

1. Build the SDK first (`swift build` in VoiceStreamSDK folder)
2. In DemoApp project settings → **General** tab
3. Under **Frameworks, Libraries, and Embedded Content**, click `+`
4. Select the built framework from `.build/debug/`

### Option 3: Embed Source Files (Quickest for Testing)

1. Drag all `.swift` files from `VoiceStreamSDK/Sources/VoiceStreamSDK/` into the DemoApp project
2. Make sure "Copy items if needed" is **unchecked**
3. Add to DemoApp target

## Troubleshooting

### "Cannot find 'VoiceStreamSDK' in scope"

**Solution:**
- Ensure SDK is added as a dependency in project settings
- Check that `import VoiceStreamSDK` is at the top of the file
- Clean and rebuild (`⇧⌘K`, then `⌘B`)

### "Package Resolution Failed"

**Solution:**
- Check internet connection (needs to download Starscream)
- File → Packages → Reset Package Caches
- File → Packages → Update to Latest Package Versions

### "No such module 'Starscream'"

**Solution:**
- The SDK depends on Starscream for WebSocket
- Xcode should download it automatically
- If not: File → Packages → Resolve Package Versions

### "Microphone Permission Denied"

**Solution:**
- Check `Info.plist` has `NSMicrophoneUsageDescription`
- Reset simulator: Device → Erase All Content and Settings
- On real device: Settings → Privacy → Microphone → Enable for app

### Build Errors in AudioCapture/AudioPlayback

**Solution:**
- Make sure you're targeting iOS 14.0+
- Check project build settings: Deployment Target = 14.0
- Import AVFoundation at the top of the files

## Testing the SDK

### Test Plan

1. **Build Test**
   - ✓ SDK builds without errors
   - ✓ Demo app builds without errors

2. **Connection Test**
   - ✓ Tap "Connect" button
   - ✓ See "Connected" status
   - ✓ Connection duration timer starts

3. **Audio Streaming Test**
   - ✓ Tap "Start Streaming"
   - ✓ Speak into microphone
   - ✓ Check "Data Sent" metric increases
   - ✓ Check "Data Received" metric increases
   - ✓ Check event log shows audio events

4. **Metrics Test**
   - ✓ Latency measurements appear
   - ✓ Average/Min/Max latency calculated
   - ✓ Quality indicator shows (Excellent/Good/Fair/Poor)

5. **Disconnect Test**
   - ✓ Tap "Stop Streaming"
   - ✓ Tap "Disconnect"
   - ✓ Status shows "Disconnected"

### Expected Behavior

**On Connect:**
- Status changes to "Connecting..." then "Connected"
- Connection duration timer starts
- Event log shows "Connected to server"

**On Start Streaming:**
- Microphone permission requested (first time)
- "Data Sent" metric starts increasing
- Event log shows "Starting audio streaming"

**On Audio Reception:**
- "Data Received" metric increases
- Latency metrics calculated
- Quality assessment appears

**On Disconnect:**
- Audio streaming stops automatically
- Connection duration resets
- Status shows "Disconnected"

## Running on Real Device

To run on a real iPhone/iPad:

1. Connect device via USB
2. In Xcode, select your device from the device menu
3. You may need to trust the computer on the device
4. You need a development team/Apple ID:
   - Xcode → Preferences → Accounts → Add Apple ID
   - In project settings → Signing & Capabilities → Select Team
5. Build and run (`⌘R`)

## Customizing the Demo

The demo app source is in `DemoApp/DemoApp/ContentView.swift`. You can:

- Change server URL in `VoiceStreamConfig`
- Modify tenant ID and name
- Adjust audio settings
- Customize UI colors and layout
- Add additional metrics

Example:
```swift
// In ContentView.swift, find initializeSDK() method
let config = VoiceStreamConfig(
    serverUrl: "wss://your-server.com/ws",  // Change this
    tenantId: "your-id",                     // Change this
    tenantName: "Your Name",                 // Change this
    enableDebugLogging: true
)
```

## Next Steps

### For SDK Development

1. **Read the Docs:**
   - [README.md](README.md) - Full documentation
   - [PROJECT_SUMMARY.md](PROJECT_SUMMARY.md) - Technical overview
   - [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) - Integration steps

2. **Explore the Code:**
   - `VoiceStreamSDK.swift` - Main SDK class
   - `WebSocketManager.swift` - Connection handling
   - `AudioCaptureManager.swift` - Microphone capture
   - `AudioPlaybackManager.swift` - Speaker playback

3. **Customize:**
   - Modify audio settings in `VoiceStreamConfig`
   - Add error handling logic
   - Implement additional features

### For Integration

1. **Copy to Your Project:**
   - Use Swift Package Manager (recommended)
   - Or copy source files directly

2. **Follow Integration Guide:**
   - See [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
   - Check [QUICK_START.md](QUICK_START.md)

3. **Test Thoroughly:**
   - Test on real devices
   - Test various network conditions
   - Test background audio
   - Test permission flows

## Comparison with Android SDK

This iOS SDK mirrors the Android SDK features:

| Feature | Android | iOS |
|---------|---------|-----|
| WebSocket | OkHttp | Starscream ✅ |
| Audio Capture | AudioRecord | AVAudioEngine ✅ |
| Audio Playback | AudioTrack | AVAudioPlayerNode ✅ |
| Auto-Reconnect | ✅ | ✅ |
| 16kHz → 24kHz | ✅ | ✅ |
| Error Handling | ✅ | ✅ |
| Demo App | ✅ | ✅ |

## Support

If you encounter issues:

1. **Check Documentation:**
   - README.md has troubleshooting section
   - INTEGRATION_GUIDE.md has common issues

2. **Check Xcode Console:**
   - Look for error messages
   - Enable debug logging in config

3. **Clean and Rebuild:**
   - `⇧⌘K` to clean
   - `⌘B` to build

4. **Reset Packages:**
   - File → Packages → Reset Package Caches
   - File → Packages → Update to Latest Package Versions

## Resources

- **Apple Developer Docs:** https://developer.apple.com/documentation/
- **Starscream (WebSocket):** https://github.com/daltoniam/Starscream
- **AVFoundation Guide:** https://developer.apple.com/av-foundation/
- **Swift Package Manager:** https://swift.org/package-manager/

## GitHub Upload Checklist

Before uploading to GitHub:

- [ ] Test that SDK builds on Mac
- [ ] Test that Demo app runs
- [ ] Verify all dependencies resolve
- [ ] Update repository URLs in Package.swift
- [ ] Update repository URLs in README.md
- [ ] Add screenshots to README (optional)
- [ ] Tag version 1.0.0

## GitHub Repository Setup

```bash
cd SDK_tenant_iOS
git init
git add .
git commit -m "Initial commit - VoiceStreamSDK iOS v1.0.0"
git branch -M main
git remote add origin https://github.com/yourusername/VoiceStreamSDK-iOS.git
git push -u origin main
git tag v1.0.0
git push --tags
```

---

**You're all set! 🚀**

Open the project in Xcode and start testing. The SDK is ready for integration and further development.

For questions or issues, refer to the comprehensive documentation in this folder.

**Happy coding!**
