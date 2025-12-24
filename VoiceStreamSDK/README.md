# VoiceStreamSDK for iOS

A powerful iOS SDK for real-time bidirectional voice streaming over WebSocket.

## Features

- 🎙️ **Real-time Audio Capture** - Record audio from microphone at 16 kHz
- 🔊 **Real-time Audio Playback** - Play received audio at 24 kHz
- 🔄 **Bidirectional Streaming** - Simultaneous capture and playback
- 🔌 **WebSocket Connection** - Reliable connection with automatic reconnection
- 🔁 **Auto-Reconnect** - Exponential backoff strategy with configurable attempts
- 💓 **Keep-Alive** - Automatic ping/pong mechanism
- 🔐 **Authentication** - Bearer token support
- 📊 **Connection States** - Track connection lifecycle
- ⚠️ **Comprehensive Error Handling** - Type-safe error reporting
- 🎯 **Simple API** - Easy-to-use singleton pattern

## Requirements

- iOS 14.0+
- Xcode 14.0+
- Swift 5.9+

## Installation

### Swift Package Manager

Add the following to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/yourusername/VoiceStreamSDK-iOS.git", from: "1.0.0")
]
```

Or in Xcode:
1. File → Add Packages...
2. Enter the repository URL
3. Select version and add to your target

### Manual Installation

1. Drag and drop the `VoiceStreamSDK` folder into your Xcode project
2. Make sure "Copy items if needed" is checked
3. Add to your target

## Permissions

Add the following to your `Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>We need access to your microphone for voice streaming</string>
```

## Quick Start

### 1. Import the SDK

```swift
import VoiceStreamSDK
```

### 2. Configure the SDK

```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://streaming-poc.smartserve.ai/ws",
    tenantId: "your-tenant-id",
    tenantName: "Your Tenant Name",
    enableDebugLogging: true
)
```

### 3. Initialize the SDK

```swift
let sdk = VoiceStreamSDK.initialize(config: config)
```

### 4. Set up callbacks

```swift
sdk.setCallback(object: self) // Your class should conform to VoiceStreamCallback

// Or use closures
sdk.onConnected = {
    print("Connected to server")
}

sdk.onMessage = { message in
    print("Received message: \(message)")
}

sdk.onAudioReceived = { audioData in
    print("Received audio: \(audioData.count) bytes")
}

sdk.onError = { error in
    print("Error: \(error)")
}

sdk.onDisconnected = { reason in
    print("Disconnected: \(reason)")
}
```

### 5. Connect and start streaming

```swift
// Connect to server
sdk.connect()

// Start audio streaming (after connection is established)
sdk.startAudioStreaming()

// Stop streaming
sdk.stopAudioStreaming()

// Disconnect
sdk.disconnect()

// Cleanup when done
sdk.cleanup()
```

## API Reference

### VoiceStreamSDK

Main SDK class (Singleton pattern).

#### Initialization

```swift
static func initialize(config: VoiceStreamConfig) -> VoiceStreamSDK
static func getInstance() throws -> VoiceStreamSDK
```

#### Connection Management

```swift
func connect()
func disconnect()
func isConnected() -> Bool
func getConnectionState() -> ConnectionState
```

#### Audio Streaming

```swift
func startAudioStreaming()
func stopAudioStreaming()
func isStreaming() -> Bool
```

#### Messaging

```swift
func sendMessage(_ text: String)
```

#### Callback Setup

```swift
func setCallback(object: VoiceStreamCallback)
// Or use closure properties
var onConnected: (() -> Void)?
var onMessage: ((String) -> Void)?
var onAudioReceived: ((Data) -> Void)?
var onAudioSent: ((Data) -> Void)?
var onError: ((VoiceStreamError) -> Void)?
var onDisconnected: ((String) -> Void)?
```

#### Lifecycle

```swift
func cleanup()
func reset() // For testing only
```

### VoiceStreamConfig

Configuration object for SDK initialization.

```swift
struct VoiceStreamConfig {
    let serverUrl: String
    let tenantId: String
    let tenantName: String
    let authToken: String?
    let autoReconnect: Bool
    let maxReconnectAttempts: Int
    let reconnectDelayMs: Int
    let maxReconnectDelayMs: Int
    let pingIntervalMs: Int
    let enableDebugLogging: Bool
    let audioInputSampleRate: Double
    let audioOutputSampleRate: Double
    let audioChannels: Int
    let audioBitDepth: Int
    let audioBufferSize: Int
}
```

**Default values:**
- `serverUrl`: "wss://streaming-poc.smartserve.ai/ws"
- `authToken`: nil
- `autoReconnect`: true
- `maxReconnectAttempts`: 5
- `reconnectDelayMs`: 1000 (1 second)
- `maxReconnectDelayMs`: 30000 (30 seconds)
- `pingIntervalMs`: 30000 (30 seconds)
- `enableDebugLogging`: false
- `audioInputSampleRate`: 16000.0 (16 kHz)
- `audioOutputSampleRate`: 24000.0 (24 kHz)
- `audioChannels`: 1 (mono)
- `audioBitDepth`: 16
- `audioBufferSize`: 1600 (100ms at 16kHz)

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

All methods have default empty implementations, so you only need to implement the ones you care about.

### VoiceStreamError

Error types that can occur during SDK operation.

```swift
enum VoiceStreamError {
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

## Advanced Usage

### Custom Audio Configuration

```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://your-server.com/ws",
    tenantId: "tenant123",
    tenantName: "My Tenant",
    audioInputSampleRate: 16000.0,
    audioOutputSampleRate: 24000.0,
    audioBufferSize: 3200 // 200ms buffer
)
```

### Authentication

```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://your-server.com/ws",
    tenantId: "tenant123",
    tenantName: "My Tenant",
    authToken: "your-bearer-token"
)
```

### Reconnection Settings

```swift
let config = VoiceStreamConfig(
    serverUrl: "wss://your-server.com/ws",
    tenantId: "tenant123",
    tenantName: "My Tenant",
    autoReconnect: true,
    maxReconnectAttempts: 10,
    reconnectDelayMs: 2000,
    maxReconnectDelayMs: 60000
)
```

## Demo App

The SDK includes a complete demo application showing all features:

- Connection management
- Audio streaming controls
- Real-time event logging
- Latency measurement
- Data transfer metrics
- Connection statistics

To run the demo:
1. Open `DemoApp/DemoApp.xcodeproj` in Xcode
2. Build and run on a device or simulator
3. Grant microphone permissions
4. Tap "Connect" to establish connection
5. Tap "Start Streaming" to begin audio capture/playback

## Architecture

The SDK uses a clean architecture with separated concerns:

- **VoiceStreamSDK**: Main facade (Singleton pattern)
- **WebSocketManager**: WebSocket connection handling
- **AudioCaptureManager**: Microphone audio capture
- **AudioPlaybackManager**: Speaker audio playback
- **VoiceStreamConfig**: Configuration management
- **VoiceStreamCallback**: Event notification protocol
- **VoiceStreamError**: Type-safe error handling
- **ConnectionState**: Connection lifecycle tracking

## Audio Flow

### Capture Path
```
Microphone → AudioCaptureManager → 16kHz PCM → WebSocket → Server
```

### Playback Path
```
Server → WebSocket → 24kHz PCM → AudioPlaybackManager → Speaker
```

## Troubleshooting

### Microphone Permission Denied

Make sure you've added `NSMicrophoneUsageDescription` to your `Info.plist` and requested permission:

```swift
import AVFoundation

AVAudioSession.sharedInstance().requestRecordPermission { granted in
    if granted {
        // Permission granted
    } else {
        // Permission denied
    }
}
```

### Connection Fails

- Check your server URL is correct
- Ensure you have internet connectivity
- Verify the server is running and accessible
- Check if authentication token is required

### Audio Not Playing

- Ensure audio streaming is started: `sdk.startAudioStreaming()`
- Check device volume
- Verify audio session configuration
- Check if audio data is being received in callbacks

### Auto-Reconnect Not Working

- Check if `autoReconnect` is enabled in config
- Verify `maxReconnectAttempts` hasn't been exceeded
- Check network connectivity
- Review error callbacks for reconnection failures

## Best Practices

1. **Initialize once**: Use the singleton pattern properly - initialize once per app lifecycle
2. **Handle permissions**: Request microphone permissions before starting audio streaming
3. **Cleanup resources**: Call `cleanup()` when done to release resources
4. **Handle errors**: Implement error callbacks to handle connection and audio errors
5. **Background audio**: Configure audio session for background operation if needed
6. **Testing**: Use the demo app as a reference implementation

## Thread Safety

The SDK handles thread safety internally:
- WebSocket operations run on dedicated queue
- Audio capture/playback run on their own threads
- Callbacks are dispatched on the main queue by default

## Performance

- **Low latency**: Optimized for real-time audio streaming
- **Memory efficient**: Minimal buffer allocation
- **Battery friendly**: Efficient audio processing
- **Network optimized**: Automatic reconnection with exponential backoff

## License

[Your License Here]

## Support

For issues, questions, or contributions, please visit:
[Your GitHub Repository URL]

## Changelog

### Version 1.0.0
- Initial release
- Bidirectional audio streaming
- WebSocket connection management
- Auto-reconnection support
- Comprehensive error handling
- Demo application
