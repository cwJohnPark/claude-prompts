---
name: autonomous-app-testing
description: |
  iOS Simulator 및 Android Emulator 자율 앱 테스트 스킬.
  앱 설치, UI 인터랙션, 스크린샷 캡처, E2E 테스트를 자율적으로 수행합니다.
category: apple
tags: [ios, android, testing, automation, simulator, emulator]
---

# Autonomous App Testing

Fully autonomous testing workflow for mobile applications on iOS Simulator and Android Emulator.

## Quick Start

1. Detect platform from app file (`.app`/`.ipa` = iOS, `.apk` = Android)
2. Launch simulator/emulator
3. Install and launch the app
4. Capture initial screenshot
5. Inspect UI hierarchy
6. Execute test scenarios autonomously
7. Generate test report

## Platform Detection

```python
import os

def detect_platform(app_path):
    ext = os.path.splitext(app_path)[1].lower()
    if ext in ['.app', '.ipa']:
        return 'ios'
    elif ext == '.apk':
        return 'android'
    raise ValueError(f"Unknown app format: {ext}")
```

## iOS Simulator Control

### List Available Simulators
```bash
xcrun simctl list devices available --json
```

### Boot Simulator
```bash
# Boot specific device by UDID
xcrun simctl boot <UDID>

# Or boot by name
xcrun simctl boot "iPhone 15 Pro"
```

### Open Simulator App (makes it visible)
```bash
open -a Simulator
```

### Install App
```bash
xcrun simctl install booted /path/to/App.app
```

### Launch App
```bash
# Get bundle ID from Info.plist
xcrun simctl launch booted <bundle_id>
```

### Capture Screenshot
```bash
xcrun simctl io booted screenshot /tmp/screenshot.png
```

### Get UI Hierarchy (Accessibility)
```bash
# Using accessibility inspector
xcrun simctl ui booted describe window
```

### Simulate Touch
```bash
# Tap at coordinates (requires applesimutils or custom approach)
xcrun simctl io booted tap <x> <y>
```

### Input Text
```bash
xcrun simctl io booted input text "Hello World"
```

### Terminate App
```bash
xcrun simctl terminate booted <bundle_id>
```

### Shutdown Simulator
```bash
xcrun simctl shutdown booted
```

## Android Emulator Control

### List Available Emulators
```bash
emulator -list-avds
```

### Start Emulator
```bash
# Start with GUI
emulator -avd <avd_name> &

# Wait for boot completion
adb wait-for-device
adb shell getprop sys.boot_completed | grep 1
```

### Install APK
```bash
adb install /path/to/app.apk
```

### Launch App
```bash
# Get package name from APK
aapt dump badging app.apk | grep package | awk '{print $2}'

# Launch
adb shell am start -n <package>/<activity>
```

### Capture Screenshot
```bash
adb exec-out screencap -p > /tmp/screenshot.png
```

### Get UI Hierarchy
```bash
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml
```

### Simulate Touch
```bash
# Tap at coordinates
adb shell input tap <x> <y>

# Long press
adb shell input swipe <x> <y> <x> <y> 1000

# Swipe
adb shell input swipe <x1> <y1> <x2> <y2> <duration_ms>
```

### Input Text
```bash
adb shell input text "Hello"
```

### Press Keys
```bash
# Back button
adb shell input keyevent KEYCODE_BACK

# Home button
adb shell input keyevent KEYCODE_HOME

# Enter
adb shell input keyevent KEYCODE_ENTER
```

### Stop App
```bash
adb shell am force-stop <package>
```

### Shutdown Emulator
```bash
adb emu kill
```

## Autonomous Testing Workflow

### Step 1: Environment Setup

```python
import subprocess
import json
import time
import os

class SimulatorController:
    def __init__(self, platform):
        self.platform = platform

    def get_available_devices(self):
        if self.platform == 'ios':
            result = subprocess.run(
                ['xcrun', 'simctl', 'list', 'devices', 'available', '--json'],
                capture_output=True, text=True
            )
            return json.loads(result.stdout)
        else:
            result = subprocess.run(
                ['emulator', '-list-avds'],
                capture_output=True, text=True
            )
            return result.stdout.strip().split('\n')
```

### Step 2: Screenshot Analysis Loop

The core autonomous testing loop:

```python
def autonomous_test_loop(controller, max_iterations=20):
    """
    Main autonomous testing loop.
    1. Capture screenshot
    2. Analyze UI (Claude vision)
    3. Decide next action
    4. Execute action
    5. Repeat until done or max iterations
    """
    results = []
    visited_screens = set()

    for i in range(max_iterations):
        # Capture current state
        screenshot_path = controller.capture_screenshot()
        ui_hierarchy = controller.get_ui_hierarchy()

        # Analyze and decide (Claude makes decision)
        # Return screenshot to Claude for visual analysis
        # Claude determines: what screen is this? what to test? what to tap?

        # Execute action based on Claude's decision
        # Record results

    return results
```

### Step 3: UI Element Detection

Parse UI hierarchy to find interactive elements:

```python
def parse_ios_accessibility(xml_content):
    """Extract tappable elements from iOS accessibility tree."""
    import xml.etree.ElementTree as ET
    elements = []
    root = ET.fromstring(xml_content)
    for elem in root.iter():
        if elem.get('accessible') == 'true':
            elements.append({
                'label': elem.get('label'),
                'type': elem.tag,
                'frame': elem.get('frame'),
                'enabled': elem.get('enabled') == 'true'
            })
    return elements

def parse_android_uiautomator(xml_content):
    """Extract tappable elements from Android UI dump."""
    import xml.etree.ElementTree as ET
    elements = []
    root = ET.fromstring(xml_content)
    for node in root.iter('node'):
        if node.get('clickable') == 'true':
            bounds = node.get('bounds')  # [x1,y1][x2,y2]
            elements.append({
                'text': node.get('text'),
                'class': node.get('class'),
                'resource_id': node.get('resource-id'),
                'bounds': bounds,
                'enabled': node.get('enabled') == 'true'
            })
    return elements
```

## Claude's Autonomous Decision Making

During testing, Claude should:

1. **Observe**: View screenshot and UI hierarchy
2. **Analyze**: Identify current screen, available actions, untested areas
3. **Plan**: Decide which element to interact with and why
4. **Act**: Execute the interaction (tap, swipe, type)
5. **Verify**: Check if expected result occurred
6. **Record**: Log the test step and result
7. **Iterate**: Move to next test case

### Decision Criteria

- Prioritize unexplored UI paths
- Test all visible interactive elements
- Verify text inputs and form submissions
- Check navigation flows (forward/back)
- Test error states (invalid inputs, network errors)
- Verify accessibility labels exist

## Test Report Generation

Generate structured test report:

```markdown
# App Test Report

## Summary
- Platform: iOS/Android
- Device: iPhone 15 Pro / Pixel 7
- App: com.example.app
- Total Screens Tested: X
- Total Interactions: Y
- Issues Found: Z

## Screen Coverage
1. Login Screen
2. Home Screen
3. Profile Screen
...

## Issues Found
### Issue 1: Button not responding
- Screen: Settings
- Element: "Save" button
- Expected: Save settings
- Actual: No response
- Screenshot: issue_1.png

## Test Steps Log
| Step | Action | Element | Result |
|------|--------|---------|--------|
| 1 | Launch | App | Success |
| 2 | Tap | Login button | Success |
...
```

## Troubleshooting

### iOS Simulator Issues

```bash
# Reset simulator
xcrun simctl erase booted

# Check simulator status
xcrun simctl list devices | grep Booted

# Reinstall corrupt app
xcrun simctl uninstall booted <bundle_id>
xcrun simctl install booted /path/to/App.app
```

### Android Emulator Issues

```bash
# Cold boot emulator
emulator -avd <n> -no-snapshot-load

# Clear app data
adb shell pm clear <package>

# Check connected devices
adb devices
```

## References

- See `11-autonomous-app-testing/testing-patterns.md` for common test scenarios
- See `11-autonomous-app-testing/ui-patterns.md` for UI interaction patterns
