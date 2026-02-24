---
name: ui-patterns
description: 자율 테스트용 좌표 및 요소 기반 UI 인터랙션 패턴
category: apple
tags: [swiftui, ios, testing, ui-patterns]
---

# UI Interaction Patterns

Coordinate-based and element-based interaction patterns for autonomous testing.

## Screen Coordinate Systems

### iOS Simulator
- Origin: Top-left (0, 0)
- Coordinates in points (not pixels)
- Common resolutions:
  - iPhone 15 Pro: 393 x 852 points
  - iPhone 15 Pro Max: 430 x 932 points
  - iPad Pro 11": 834 x 1194 points

### Android Emulator
- Origin: Top-left (0, 0)
- Coordinates in pixels
- Common resolutions:
  - Pixel 7: 1080 x 2400 pixels
  - Pixel Tablet: 1600 x 2560 pixels

## Common UI Zones

```
┌─────────────────────────────┐
│      Status Bar (50px)       │
├─────────────────────────────┤
│     Navigation Bar (44pt)    │
│     ← Title          ⋯      │
├─────────────────────────────┤
│                             │
│                             │
│      Content Area           │
│                             │
│                             │
├─────────────────────────────┤
│     Tab Bar / Bottom Nav    │
│     (49pt iOS / 56dp AND)   │
└─────────────────────────────┘
```

### Safe Tap Zones by Screen Region
| Region | iOS Y Range | Android Y Range |
|--------|-------------|-----------------|
| Status bar | 0-50 | 0-50 |
| Nav bar | 50-94 | 50-106 |
| Content | 94-803 | 106-2344 |
| Tab bar | 803-852 | 2344-2400 |

## Element Detection from Screenshots

### Visual Cues for Buttons
- Rounded rectangles with contrasting color
- Text with action words (Submit, Next, Login, Save)
- Icons within tappable areas
- Raised/shadow effects

### Visual Cues for Input Fields
- Outlined rectangles
- Placeholder text (grayed out)
- Text cursor when focused
- Keyboard presence indicates active input

### Visual Cues for Lists
- Repeating similar elements
- Chevrons (>) indicating drill-down
- Divider lines between items

## Bounds Parsing (Android)

Android UI automator returns bounds in format: `[x1,y1][x2,y2]`

```python
import re

def parse_bounds(bounds_str):
    """Parse Android bounds string to coordinates."""
    match = re.findall(r'\[(\d+),(\d+)\]', bounds_str)
    if len(match) == 2:
        x1, y1 = int(match[0][0]), int(match[0][1])
        x2, y2 = int(match[1][0]), int(match[1][1])
        return {
            'x1': x1, 'y1': y1,
            'x2': x2, 'y2': y2,
            'center_x': (x1 + x2) // 2,
            'center_y': (y1 + y2) // 2,
            'width': x2 - x1,
            'height': y2 - y1
        }
    return None

def tap_element(bounds_str):
    """Get tap coordinates for an element."""
    coords = parse_bounds(bounds_str)
    if coords:
        return coords['center_x'], coords['center_y']
    return None
```

## Gesture Patterns

### Tap
```bash
# Android
adb shell input tap X Y

# iOS
xcrun simctl io booted tap X Y
```

### Double Tap
```bash
# Android
adb shell input tap X Y && sleep 0.1 && adb shell input tap X Y
```

### Long Press
```bash
# Android (swipe with same start/end = long press)
adb shell input swipe X Y X Y 1000  # 1000ms = 1 second
```

### Swipe/Scroll
```bash
# Android - Scroll down (content moves up)
adb shell input swipe 540 1500 540 500 300

# Android - Scroll up (content moves down)
adb shell input swipe 540 500 540 1500 300

# Android - Swipe left (like next page)
adb shell input swipe 900 1000 100 1000 200

# Android - Swipe right (like previous page)
adb shell input swipe 100 1000 900 1000 200
```

### Pinch to Zoom
```bash
# Android (requires sendevent, complex)
# Alternative: use two-finger gesture via ADB
```

### Pull to Refresh
```bash
# Android - Swipe down from top of content area
adb shell input swipe 540 400 540 1200 500
```

## Text Input Strategies

### Clearing Existing Text
```bash
# Android - Select all and delete
adb shell input keyevent KEYCODE_MOVE_END  # Go to end
adb shell input keyevent --longpress KEYCODE_DEL  # Delete back

# Or use triple tap to select all, then type
adb shell input tap X Y
adb shell input tap X Y
adb shell input tap X Y
adb shell input keyevent KEYCODE_DEL
```

### Special Characters
```bash
# Android - URL encode special chars
# Space = %s
adb shell input text "hello%sworld"  # "hello world"

# Or use keyboard events
adb shell input keyevent KEYCODE_SPACE
```

### Key Events Reference
| Key | Keycode |
|-----|---------|
| Enter | KEYCODE_ENTER |
| Backspace | KEYCODE_DEL |
| Tab | KEYCODE_TAB |
| Escape | KEYCODE_ESCAPE |
| Space | KEYCODE_SPACE |
| Home | KEYCODE_HOME |
| Back | KEYCODE_BACK |
| Menu | KEYCODE_MENU |

## Dialog Handling

### Common Dialog Buttons
| Dialog Type | Button Positions |
|-------------|------------------|
| Alert (2 buttons) | Left: Cancel, Right: OK |
| Permission | Left: Deny, Right: Allow |
| Confirmation | Often centered buttons |

### Permission Dialog Handling (Android)
```bash
# Grant all permissions automatically
adb shell pm grant <package> <permission>

# Example
adb shell pm grant com.app.test android.permission.CAMERA

# Deny (revoke)
adb shell pm revoke <package> <permission>
```

### Permission Dialog Handling (iOS)
```bash
# Grant via simctl (limited support)
xcrun simctl privacy booted grant <permission> <bundle_id>

# Example
xcrun simctl privacy booted grant photos com.app.test
```

## Waiting Strategies

### After Actions
| Action | Recommended Wait |
|--------|-----------------|
| Tap button | 0.5-1s |
| Screen transition | 1-2s |
| Network request | 2-5s |
| App launch | 3-5s |
| Dialog appearance | 0.5s |

### Wait for Element
```python
import time

def wait_for_element(controller, text_or_id, timeout=10):
    """Wait for element to appear in UI hierarchy."""
    start = time.time()
    while time.time() - start < timeout:
        elements = controller.get_ui_elements()
        for elem in elements:
            if text_or_id in (elem.get('text', ''), 
                             elem.get('resource_id', ''),
                             elem.get('content_desc', '')):
                return elem
        time.sleep(0.5)
    return None
```

## Screen State Detection

### Indicators
| State | Detection Method |
|-------|-----------------|
| Loading | Spinner/progress bar visible |
| Error | Red text, alert icons |
| Success | Green checkmarks, success messages |
| Empty | "No items" text, empty state illustrations |
| Logged out | Login screen visible |

### Screen Identification
```python
def identify_screen(elements, screenshot_analysis):
    """Identify current screen based on elements and visual analysis."""
    
    # Check for common screen types
    indicators = {
        'login': ['login', 'sign in', 'email', 'password'],
        'home': ['home', 'feed', 'dashboard'],
        'settings': ['settings', 'preferences', 'account'],
        'profile': ['profile', 'my account', 'edit profile'],
        'search': ['search', 'find'],
        'detail': ['back', 'share', 'details']
    }
    
    element_texts = [e.get('text', '').lower() for e in elements]
    element_texts += [e.get('content_desc', '').lower() for e in elements]
    
    for screen_type, keywords in indicators.items():
        if any(kw in ' '.join(element_texts) for kw in keywords):
            return screen_type
    
    return 'unknown'
```

## Coordinate Estimation from Visual Analysis

When Claude analyzes screenshots, estimate tap coordinates:

1. **Identify element visually** - Note position relative to screen
2. **Estimate percentage** - Element at ~30% from left, ~60% from top
3. **Calculate coordinates**:
   - For 1080x2400 screen: x = 1080 * 0.30 = 324, y = 2400 * 0.60 = 1440
4. **Tap center of element** - Add half of estimated element size

### Safe Coordinate Offsets
- Add 10-20px buffer from edges
- Prefer center of visible elements
- Avoid status bar and navigation areas
