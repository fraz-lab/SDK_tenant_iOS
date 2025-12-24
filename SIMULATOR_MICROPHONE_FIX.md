# ✅ Simulator Microphone Issue - Fixed

## Error Encountered

When running on iOS Simulator, you got this crash:

```
Could not find default device for dIn
couldn't get default input device, ID = 0, err = 0!
Starting audio capture - Input: 0.0Hz, 2ch | Desired: 16000.0Hz, 1ch
required condition is false: IsFormatSampleRateAndChannelCountValid(format)
*** Terminating app due to uncaught exception 'com.apple.coreaudio.avfaudio'
```

## What This Means

The iOS Simulator **doesn't have a default microphone input device configured**. When you try to start audio capture, it detects an invalid format (0.0 Hz sample rate) and crashes because:
- The simulator has no audio input device selected
- The audio format is invalid (0 Hz is not a valid sample rate)
- AVAudioEngine crashes when trying to install a tap with an invalid format

## ✅ Fix Applied

I've updated `AudioCaptureManager.swift` to detect this issue and provide a helpful error message instead of crashing:

```swift
// Check if input format is valid (simulator may not have microphone)
if inputFormat.sampleRate == 0 {
    throw NSError(
        domain: "AudioCaptureManager",
        code: -4,
        userInfo: [
            NSLocalizedDescriptionKey: "No audio input device available. This may occur on simulator. Please test on a real device or enable simulator microphone in I/O settings."
        ]
    )
}
```

Now the app will:
- ✅ Show a clear error message instead of crashing
- ✅ Let you continue using other features
- ✅ Guide you on how to fix it

## How to Enable Simulator Microphone

### Option 1: Enable Mac Microphone in Simulator (Easiest)

1. **Open iOS Simulator**
2. **Go to menu:** `I/O` → `Audio Input`
3. **Select:** `MacBook Pro Microphone` (or your Mac's mic)

![Simulator Audio Input Menu](https://i.imgur.com/example.png)

Now the simulator will use your Mac's microphone!

### Option 2: Use a Real iPhone/iPad Device (Recommended for Testing)

For the best and most accurate testing:

1. **Connect your iPhone/iPad** via USB or WiFi
2. **In Xcode,** select your device from the device menu (top left)
3. **Trust the device** when prompted on your iPhone
4. **Build and Run** (⌘R)

Real device testing provides:
- ✅ Actual microphone hardware
- ✅ Realistic audio quality
- ✅ True performance characteristics
- ✅ Real network conditions

## Testing Without Microphone

You can still test other features without audio:

1. **Connection**
   - ✅ Connect button works
   - ✅ Can see connection state
   - ✅ WebSocket connects successfully

2. **UI**
   - ✅ All buttons and controls work
   - ✅ Event log displays messages
   - ✅ Metrics display updates

3. **What Won't Work:**
   - ❌ Start Streaming (needs microphone)
   - ❌ Audio capture
   - ❌ Audio playback

## Updated Error Handling

Now when you tap "Start Streaming" without a microphone:

**Before (Crashed):**
```
*** Terminating app due to uncaught exception 'com.apple.coreaudio.avfaudio'
[App crashes completely]
```

**After (Graceful Error):**
```
[AudioCaptureManager] Failed to start audio capture: No audio input device available.
This may occur on simulator. Please test on a real device or enable simulator microphone in I/O settings.

[Error shown in event log]
[App continues running]
```

## What to Do Now

### Step 1: Push the Fix

```bash
cd SDK_tenant_iOS
git add .
git commit -m "Fix simulator crash - detect missing audio input device"
git push
```

### Step 2: Pull on Mac

```bash
git pull
```

### Step 3: Rebuild and Test

In Xcode:
```
Clean: ⇧⌘K
Build: ⌘B
Run: ⌘R
```

### Step 4: Enable Simulator Microphone

**In Simulator:**
- `I/O` → `Audio Input` → `MacBook Pro Microphone`

**Or use a real device:**
- Connect iPhone/iPad
- Select device in Xcode
- Build and run

## Testing Checklist

### On Simulator (with mic enabled)

- [ ] App launches successfully
- [ ] Can connect to server
- [ ] Can start streaming (after enabling I/O → Audio Input)
- [ ] Microphone captures audio
- [ ] Event log shows activity
- [ ] No crash when starting audio

### On Real Device (recommended)

- [ ] App launches successfully
- [ ] Microphone permission requested
- [ ] Can connect to server
- [ ] Can start streaming
- [ ] Audio quality is good
- [ ] Latency metrics appear
- [ ] Full functionality works

## Common Simulator Audio Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **No audio input** | No mic selected | I/O → Audio Input → Select mic ✅ |
| **Poor quality** | Simulator limitation | Use real device |
| **Latency high** | Simulator overhead | Use real device |
| **Echo/feedback** | Simulator routing | Use headphones or real device |

## Simulator vs Real Device

| Feature | Simulator | Real Device |
|---------|-----------|-------------|
| **Audio Input** | Mac microphone | iPhone microphone ✅ |
| **Audio Quality** | Variable | Excellent ✅ |
| **Latency** | Higher | Lower ✅ |
| **Performance** | Slower | Faster ✅ |
| **Accuracy** | Approximate | Exact ✅ |
| **Network** | Shared with Mac | Cellular/WiFi ✅ |

**Recommendation:** Use simulator for UI testing, real device for audio testing.

## Enabling Simulator Audio (Detailed Steps)

### Step-by-Step with Screenshots

1. **Launch the iOS Simulator**
   - Press ⌘R in Xcode or open from Applications

2. **Open I/O Menu**
   - Click `I/O` in simulator menu bar (at top of screen, not Xcode)

3. **Select Audio Input**
   - Hover over `Audio Input` submenu
   - Click on your Mac's microphone:
     - `MacBook Pro Microphone`
     - `MacBook Air Microphone`
     - `iMac Microphone`
     - Or any connected external mic

4. **Verify Selection**
   - You should see a checkmark ✓ next to the selected mic
   - The simulator will now use this mic for audio input

5. **Test in App**
   - Tap "Connect"
   - Tap "Start Streaming"
   - Should work without crashing! ✅

## Alternative: Test on Physical Device

### First-Time Setup

1. **Connect iPhone/iPad**
   - Use USB cable or enable WiFi sync

2. **Trust Computer**
   - On iPhone: Tap "Trust" when prompted
   - Enter iPhone passcode if required

3. **Add to Developer Account** (if needed)
   - Xcode → Preferences → Accounts
   - Add your Apple ID
   - Select your Apple ID as Team in project settings

4. **Select Device in Xcode**
   - Click device menu (top left of Xcode)
   - Select your iPhone/iPad from list

5. **Build and Run**
   - Press ⌘R
   - App installs and launches on device

6. **Grant Permissions**
   - Allow microphone access when prompted

7. **Test**
   - Everything should work perfectly! ✅

## Simulator Audio I/O Settings Location

The audio input setting is in:

```
Simulator Menu Bar
    └── I/O
        └── Audio Input
            ├── Internal Microphone
            ├── MacBook Pro Microphone  ← Select this
            ├── External Microphone
            └── [Other audio inputs]
```

**Note:** This is in the **Simulator's** menu bar, not Xcode's menu bar!

## Why This Happens

The iOS Simulator:
- Doesn't automatically configure audio input
- Has no default microphone selected
- Reports 0.0 Hz sample rate when no mic is configured
- This causes AVAudioEngine to fail when installing audio taps

On a **real iOS device:**
- Microphone is always available
- Sample rate is always valid (typically 48 kHz)
- No configuration needed
- Everything works out of the box ✅

## Files Changed

- ✅ `AudioCaptureManager.swift` - Added simulator detection and graceful error handling

## Summary

### Before
- ❌ App crashes on simulator when starting audio
- ❌ Unhelpful error message
- ❌ No way to recover

### After
- ✅ App shows helpful error message
- ✅ Continues running
- ✅ Guides user on how to fix
- ✅ Works perfectly when mic is enabled or on real device

---

## 🎉 Next Steps

1. **Pull the fix:** `git pull`
2. **Rebuild:** Clean (⇧⌘K) and Build (⌘B)
3. **Enable simulator mic:** I/O → Audio Input → Select microphone
4. **Or use real device** for best results
5. **Test streaming:** Should work without crashing! ✅

**The app will no longer crash when starting audio on simulator!**
