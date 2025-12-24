# VoiceStreamSDK for iOS

<p align="center">
  <strong>Real-time bidirectional voice streaming SDK for iOS</strong>
</p>

<p align="center">
  A powerful, easy-to-use iOS SDK for real-time voice streaming over WebSocket with automatic reconnection, audio capture, and playback.
</p>

---

## Features

- 🎙️ **Real-time Audio Capture** - Record from microphone at 16 kHz, 16-bit PCM, mono
- 🔊 **Real-time Audio Playback** - Play received audio at 24 kHz with automatic volume amplification
- 🔄 **Bidirectional Streaming** - Simultaneous capture and playback
- 🔌 **WebSocket Connection** - Reliable WebSocket client with Starscream
- 🔁 **Auto-Reconnect** - Exponential backoff strategy with configurable attempts
- 💓 **Keep-Alive** - Automatic ping/pong mechanism every 30 seconds
- 🔐 **Authentication** - Bearer token support for secure connections
- 📊 **Connection States** - Track connection lifecycle (disconnected, connecting, connected, reconnecting)
- ⚠️ **Comprehensive Error Handling** - Type-safe error reporting with 10 error types
- 🎯 **Simple API** - Easy-to-use singleton pattern with both protocol and closure-based callbacks
- 📱 **SwiftUI & UIKit** - Full support for both UI frameworks
- 🔧 **Configurable** - Extensive configuration options for audio and connection settings
- 🧪 **Demo App Included** - Complete reference implementation with metrics tracking

## Requirements

| Requirement | Version |
|-------------|---------|
| **iOS** | 14.0+ |
| **Xcode** | 14.0+ |
| **Swift** | 5.9+ |
| **Dependencies** | Starscream 4.0+ |

## Installation

### Swift Package Manager (Recommended)

Add to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/yourusername/VoiceStreamSDK-iOS.git", from: "1.0.0")
]
```

Or in Xcode:
1. File → Add Packages...
2. Enter repository URL
3. Select version and add to target

### Manual Installation

1. Clone or download this repository
2. Drag `VoiceStreamSDK` folder into your Xcode project
3. Ensure "Copy items if needed" is checked
4. Add to your target

## Quick Start

### 1. Add Permissions

Add to your `Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>We need access to your microphone for voice streaming</string>
```

### 2. Import and Initialize

```swift
import VoiceStreamSDK

let config = VoiceStreamConfig(
    serverUrl: "wss://streaming-poc.smartserve.ai/ws",
    tenantId: "your-tenant-id",
    tenantName: "Your Tenant Name",
    enableDebugLogging: true
)

let sdk = VoiceStreamSDK.initialize(config: config)
```

### 3. Set Up Callbacks

```swift
sdk.onConnectedHandler = {
    print("Connected!")
    sdk.startAudioStreaming()
}

sdk.onAudioReceivedHandler = { audioData in
    print("Received \(audioData.count) bytes")
}

sdk.onErrorHandler = { error in
    print("Error: \(error)")
}
```

### 4. Connect and Stream

```swift
sdk.connect()                  // Connect to server
// sdk.startAudioStreaming()   // Start after connection
// sdk.stopAudioStreaming()    // Stop streaming
// sdk.disconnect()            // Disconnect
// sdk.cleanup()               // Cleanup resources
```

## Documentation

- **[Quick Start Guide](QUICK_START.md)** - Get started in 5 minutes
- **[Integration Guide](INTEGRATION_GUIDE.md)** - Complete step-by-step integration
- **[API Reference](#api-reference)** - Full API documentation (below)
- **[Demo App](DemoApp/)** - Complete reference implementation

## Project Structure

```
SDK_tenant_iOS/
├── VoiceStreamSDK/              # Main SDK (Swift Package)
│   ├── Package.swift            # Swift Package Manager config
│   ├── Sources/
│   │   └── VoiceStreamSDK/
│   │       ├── VoiceStreamSDK.swift          # Main SDK facade
│   │       ├── VoiceStreamConfig.swift       # Configuration
│   │       ├── VoiceStreamCallback.swift     # Event callbacks
│   │       ├── VoiceStreamError.swift        # Error types
│   │       ├── ConnectionState.swift         # Connection states
│   │       ├── WebSocketManager.swift        # WebSocket handling
│   │       ├── AudioCaptureManager.swift     # Microphone capture
│   │       └── AudioPlaybackManager.swift    # Speaker playback
│   └── Tests/
│       └── VoiceStreamSDKTests/
│
├── DemoApp/                     # Demo Application
│   ├── DemoApp.xcodeproj        # Xcode project
│   └── DemoApp/
│       ├── DemoAppApp.swift     # App entry point
│       ├── ContentView.swift    # Main UI (SwiftUI)
│       ├── Info.plist           # App permissions
│       └── Assets.xcassets/     # App assets
│
├── README.md                    # This file
├── QUICK_START.md              # Quick start guide
└── INTEGRATION_GUIDE.md        # Detailed integration guide
```

## Architecture

The SDK follows a clean architecture with separated concerns:

```
┌─────────────────────────────────────────┐
│         Your Application                │
│  (SwiftUI / UIKit View Controller)      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│        VoiceStreamSDK (Facade)          │
│  - Singleton Pattern                    │
│  - Callback Management                  │
│  - Lifecycle Coordination               │
└─────┬──────────┬─────────────┬──────────┘
      │          │             │
      ▼          ▼             ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│WebSocket │ │  Audio   │ │    Audio     │
│ Manager  │ │ Capture  │ │  Playback    │
│          │ │ Manager  │ │   Manager    │
└──────────┘ └──────────┘ └──────────────┘
      │          │             │
      ▼          ▼             ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│Starscream│ │AudioRecord│ │  AudioTrack  │
│WebSocket │ │(16kHz PCM)│ │ (24kHz PCM)  │
└──────────┘ └──────────┘ └──────────────┘
      │          │             │
      ▼          ▼             ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│  Server  │ │Microphone│ │   Speaker    │
└──────────┘ └──────────┘ └──────────────┘
```

### Audio Flow

**Capture Path (Microphone → Server):**
```
Microphone → AVAudioEngine → AudioCaptureManager
         ↓
    16kHz PCM Frames (Int16)
         ↓
  Format Conversion (if needed)
         ↓
  WebSocketManager.sendBinary()
         ↓
   Starscream WebSocket
         ↓
      Server
```

**Playback Path (Server → Speaker):**
```
Server
   ↓
Starscream WebSocket
   ↓
WebSocketManager.onBinary()
   ↓
VoiceStreamSDK.onAudioReceived()
   ↓
AudioPlaybackManager.queueAudio()
   ↓
Audio Queue (DispatchQueue)
   ↓
Volume Amplification (3.0x)
   ↓
AVAudioPlayerNode.scheduleBuffer()
   ↓
AVAudioEngine → Speaker
```

## API Reference

### VoiceStreamSDK

Main SDK class (Singleton pattern).

#### Initialization

```swift
static func initialize(config: VoiceStreamConfig) -> VoiceStreamSDK
static func getInstance() throws -> VoiceStreamSDK
static func reset()  // Testing only
```

**Example:**
```swift
let config = VoiceStreamConfig(tenantId: "id", tenantName: "name")
let sdk = VoiceStreamSDK.initialize(config: config)

// Later
let sdk = try VoiceStreamSDK.getInstance()
```

#### Callback Setup

```swift
func setCallback(object: VoiceStreamCallback)

// Or use closures
var onConnected: (() -> Void)?
var onMessage: ((String) -> Void)?
var onAudioReceived: ((Data) -> Void)?
var onAudioSent: ((Data) -> Void)?
var onError: ((VoiceStreamError) -> Void)?
var onDisconnected: ((String) -> Void)?
```

**Example:**
```swift
// Protocol-based
sdk.setCallback(object: self)

// Closure-based
sdk.onConnectedHandler = { print("Connected") }
sdk.onErrorHandler = { error in print("Error: \(error)") }
```

#### Connection Management

```swift
func connect()
func disconnect()
func isConnected() -> Bool
func getConnectionState() -> ConnectionState
```

**Example:**
```swift
sdk.connect()

if sdk.isConnected() {
    print("Connected!")
}

let state = sdk.getConnectionState()  // .connected, .connecting, etc.
```

#### Audio Streaming

```swift
func startAudioStreaming()
func stopAudioStreaming()
func isStreaming() -> Bool
```

**Example:**
```swift
sdk.startAudioStreaming()  // Start capture + playback

if sdk.isStreaming() {
    print("Currently streaming")
}

sdk.stopAudioStreaming()  // Stop capture + playback
```

#### Messaging

```swift
func sendMessage(_ text: String)
```

**Example:**
```swift
sdk.sendMessage("Hello from iOS!")
```

#### Lifecycle

```swift
func cleanup()
```

**Example:**
```swift
deinit {
    sdk.cleanup()
}
```

### VoiceStreamConfig

Configuration object for SDK.

```swift
struct VoiceStreamConfig {
    // Server
    let serverUrl: String                    // Default: "wss://streaming-poc.smartserve.ai/ws"
    let tenantId: String                     // Required
    let tenantName: String                   // Required
    let authToken: String?                   // Optional Bearer token

    // Connection
    let autoReconnect: Bool                  // Default: true
    let maxReconnectAttempts: Int            // Default: 5
    let reconnectDelayMs: Int                // Default: 1000 (1s)
    let maxReconnectDelayMs: Int             // Default: 30000 (30s)
    let pingIntervalMs: Int                  // Default: 30000 (30s)

    // Audio Input (Microphone)
    let audioInputSampleRate: Double         // Default: 16000.0 (16kHz)
    let audioChannels: Int                   // Default: 1 (mono)
    let audioBitDepth: Int                   // Default: 16
    let audioBufferSize: Int                 // Default: 1600 (100ms)

    // Audio Output (Speaker)
    let audioOutputSampleRate: Double        // Default: 24000.0 (24kHz)

    // Debug
    let enableDebugLogging: Bool             // Default: false
}
```

**Example:**
```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://your-server.com/ws",
    tenantId: "tenant123",
    tenantName: "My Tenant",
    authToken: "your-jwt-token",
    autoReconnect: true,
    maxReconnectAttempts: 10,
    enableDebugLogging: true
)
```

### VoiceStreamCallback

Protocol for receiving SDK events.

```swift
protocol VoiceStreamCallback: AnyObject {
    func onConnected()
    func onMessage(message: String)
    func onAudioReceived(audioData: Data)
    func onAudioSent(audioData: Data)
    func onError(error: VoiceStreamError)
    func onDisconnected(reason: String)
}
```

All methods have default implementations, implement only what you need.

**Example:**
```swift
extension MyViewController: VoiceStreamCallback {
    func onConnected() {
        print("Connected")
    }

    func onError(error: VoiceStreamError) {
        print("Error: \(error)")
    }

    // Other methods optional
}
```

### VoiceStreamError

Error enumeration with associated values.

```swift
enum VoiceStreamError: Error {
    case connectionFailed(String)
    case authenticationFailed(String)
    case disconnected(String)
    case reconnectionFailed(String)
    case audioCaptureFailed(String)
    case audioPlaybackFailed(String)
    case audioPermissionDenied(String)
    case invalidMessage(String)
    case messageSendFailed(String)
    case unknown(String)
}
```

**Example:**
```swift
sdk.onErrorHandler = { error in
    switch error {
    case .audioPermissionDenied(let msg):
        showPermissionAlert()
    case .connectionFailed(let msg):
        showConnectionError(msg)
    default:
        print("Error: \(error.localizedDescription)")
    }
}
```

### ConnectionState

Connection state enumeration.

```swift
enum ConnectionState {
    case disconnected
    case connecting
    case connected
    case reconnecting
}
```

**Example:**
```swift
let state = sdk.getConnectionState()

switch state {
case .connected:
    print("Ready to stream")
case .connecting, .reconnecting:
    print("Please wait...")
case .disconnected:
    print("Not connected")
}
```

## Usage Examples

### SwiftUI App

```swift
import SwiftUI
import VoiceStreamSDK

class StreamViewModel: ObservableObject {
    @Published var isConnected = false
    @Published var statusMessage = "Disconnected"

    private var sdk: VoiceStreamSDK?

    init() {
        let config = VoiceStreamConfig(
            tenantId: "demo",
            tenantName: "Demo App"
        )

        sdk = VoiceStreamSDK.initialize(config: config)

        sdk?.onConnected = { [weak self] in
            DispatchQueue.main.async {
                self?.isConnected = true
                self?.statusMessage = "Connected"
            }
        }

        sdk?.onDisconnected = { [weak self] reason in
            DispatchQueue.main.async {
                self?.isConnected = false
                self?.statusMessage = "Disconnected: \(reason)"
            }
        }
    }

    func connect() { sdk?.connect() }
    func startStreaming() { sdk?.startAudioStreaming() }
    func stop() {
        sdk?.stopAudioStreaming()
        sdk?.disconnect()
    }
}

struct ContentView: View {
    @StateObject var viewModel = StreamViewModel()

    var body: some View {
        VStack {
            Text(viewModel.statusMessage)

            Button("Connect") { viewModel.connect() }
            Button("Start") { viewModel.startStreaming() }
                .disabled(!viewModel.isConnected)
            Button("Stop") { viewModel.stop() }
        }
    }
}
```

### UIKit App

```swift
import UIKit
import VoiceStreamSDK

class ViewController: UIViewController, VoiceStreamCallback {
    private var sdk: VoiceStreamSDK?

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = VoiceStreamConfig(
            tenantId: "demo",
            tenantName: "Demo"
        )

        sdk = VoiceStreamSDK.initialize(config: config)
        sdk?.setCallback(object: self)
    }

    @IBAction func connectTapped(_ sender: UIButton) {
        sdk?.connect()
    }

    @IBAction func startStreamingTapped(_ sender: UIButton) {
        sdk?.startAudioStreaming()
    }

    // MARK: - VoiceStreamCallback

    func onConnected() {
        print("Connected")
        // Update UI
    }

    func onError(error: VoiceStreamError) {
        print("Error: \(error)")
        // Show alert
    }

    deinit {
        sdk?.cleanup()
    }
}
```

## Advanced Features

### Custom Audio Configuration

```swift
let config = VoiceStreamConfig(
    tenantId: "id",
    tenantName: "name",
    audioInputSampleRate: 16000.0,   // 16kHz mic
    audioOutputSampleRate: 24000.0,  // 24kHz speaker
    audioBufferSize: 3200            // 200ms buffer for lower latency
)
```

### Background Audio

Add to `Info.plist`:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
</array>
```

Configure audio session:

```swift
import AVFoundation

let session = AVAudioSession.sharedInstance()
try? session.setCategory(.playAndRecord, mode: .voiceChat, options: [.defaultToSpeaker])
try? session.setActive(true)
```

### Authentication

```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://secure-server.com/ws",
    tenantId: "id",
    tenantName: "name",
    authToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
)
```

SDK adds `Authorization: Bearer <token>` header automatically.

## Demo App

The included demo app shows all features:

- ✅ Connection management
- ✅ Audio streaming controls
- ✅ Real-time event logging
- ✅ Latency measurement
- ✅ Data transfer metrics
- ✅ Connection duration tracking
- ✅ Quality assessment

**To run:**
1. Open `DemoApp/DemoApp.xcodeproj`
2. Build and run (⌘R)
3. Grant microphone permission
4. Tap "Connect" → "Start Streaming"

## Troubleshooting

### Permission Denied

**Problem:** Microphone permission not granted

**Solution:**
- Add `NSMicrophoneUsageDescription` to Info.plist
- Request permission: `AVAudioSession.sharedInstance().requestRecordPermission`
- Check Settings → Privacy → Microphone

### Connection Fails

**Problem:** Cannot connect to server

**Solution:**
- Verify server URL (must start with `wss://` or `ws://`)
- Check internet connectivity
- Enable debug logging: `enableDebugLogging: true`
- Check firewall/network restrictions

### No Audio

**Problem:** No audio playback

**Solution:**
- Ensure device is not muted
- Check `startAudioStreaming()` was called
- Verify audio session configuration
- Check if audio data received in `onAudioReceived` callback

### Build Errors

**Problem:** SDK won't build

**Solution:**
```bash
# Clean build
⇧⌘K (Product → Clean Build Folder)

# Update packages
File → Packages → Update to Latest Package Versions

# Reset caches
File → Packages → Reset Package Caches
```

## Performance

- **Low Latency:** Optimized for real-time streaming (< 200ms typical)
- **Memory Efficient:** Minimal buffer allocation, automatic cleanup
- **Battery Friendly:** Efficient audio processing, configurable buffer sizes
- **Network Optimized:** Automatic reconnection with exponential backoff

## Best Practices

1. **Initialize once:** Use singleton pattern properly
2. **Request permissions early:** Before connecting
3. **Handle errors:** Implement error callbacks
4. **Cleanup resources:** Call `cleanup()` when done
5. **Use weak self:** In closures to avoid retain cycles
6. **Test network conditions:** Handle reconnections gracefully

## License

[Your License Here]

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

- **Issues:** [GitHub Issues](https://github.com/yourusername/VoiceStreamSDK-iOS/issues)
- **Documentation:** Check the guides in this repository
- **Email:** support@smartserve.ai

## Changelog

### Version 1.0.0
- Initial release
- Bidirectional audio streaming (16kHz capture, 24kHz playback)
- WebSocket connection with auto-reconnection
- Comprehensive error handling
- SwiftUI and UIKit support
- Demo application with metrics
- Complete documentation

---

**Made with ❤️ for iOS developers**
