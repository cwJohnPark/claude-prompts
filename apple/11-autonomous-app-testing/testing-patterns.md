---
name: testing-patterns
description: 자율 앱 테스트 시나리오 및 패턴 참조 문서
category: apple
tags: [swiftui, ios, testing, patterns]
---

# Testing Patterns Reference

Common test scenarios and patterns for autonomous app testing.

## Table of Contents
1. [Login/Authentication Flow](#login-authentication-flow)
2. [Navigation Testing](#navigation-testing)
3. [Form Input Testing](#form-input-testing)
4. [List/Scroll Testing](#list-scroll-testing)
5. [Error State Testing](#error-state-testing)
6. [Accessibility Testing](#accessibility-testing)
7. [Performance Indicators](#performance-indicators)

## Login/Authentication Flow

### Test Scenarios
| Scenario | Input | Expected Result |
|----------|-------|-----------------|
| Valid login | correct credentials | Navigate to home |
| Empty email | empty + password | Show error |
| Empty password | email + empty | Show error |
| Invalid email format | "notanemail" | Show validation error |
| Wrong password | email + wrong pass | Show auth error |
| Forgot password | tap link | Navigate to recovery |

### Execution Pattern
```
1. Screenshot login screen
2. Identify email/username field
3. Tap field, input test email
4. Identify password field
5. Tap field, input test password
6. Find and tap login button
7. Wait 2-3 seconds
8. Screenshot result
9. Verify: new screen OR error message
```

## Navigation Testing

### Test Scenarios
- Tab bar navigation (tap each tab)
- Hamburger menu navigation
- Back button behavior
- Deep link handling
- Gesture navigation (swipe to go back)

### Execution Pattern
```
1. Identify navigation elements (tabs, menu icons)
2. Screenshot current screen
3. Tap navigation element
4. Wait for transition
5. Screenshot new screen
6. Verify: correct destination
7. Navigate back
8. Verify: return to original
```

### Common Navigation Elements
| Platform | Element | Action |
|----------|---------|--------|
| iOS | Back button (< or arrow) | Tap to go back |
| iOS | Tab bar | Tap icons at bottom |
| Android | Back button (system) | adb keyevent BACK |
| Android | Bottom navigation | Tap icons |
| Both | Hamburger menu (≡) | Tap to open drawer |

## Form Input Testing

### Test Data Sets
```python
TEST_DATA = {
    'valid_email': 'test@example.com',
    'invalid_email': 'notanemail',
    'valid_phone': '+1234567890',
    'special_chars': "Test'Name\"<script>",
    'unicode': '테스트 测试 🎉',
    'long_text': 'A' * 500,
    'sql_injection': "'; DROP TABLE users;--",
    'xss': '<script>alert(1)</script>'
}
```

### Input Validation Tests
1. Required field empty → error shown
2. Min length violation → error shown
3. Max length → truncated or error
4. Invalid format → validation error
5. Special characters → handled safely
6. Unicode characters → displayed correctly

### Execution Pattern
```
1. Identify input field by hint/label
2. Tap to focus
3. Clear existing text (if any)
4. Input test data
5. Tap outside or next field
6. Check for validation message
7. Screenshot result
```

## List/Scroll Testing

### Test Scenarios
- Scroll to load more (infinite scroll)
- Pull to refresh
- Empty state display
- Loading indicators
- Item tap navigation

### Scroll Commands
```bash
# Android - Scroll down
adb shell input swipe 500 1500 500 500 300

# Android - Scroll up
adb shell input swipe 500 500 500 1500 300

# iOS - Similar via simctl (if available)
```

### Execution Pattern
```
1. Screenshot initial list
2. Count visible items
3. Scroll down
4. Verify new items loaded
5. Continue scrolling to test pagination
6. Pull to refresh (swipe down from top)
7. Verify refresh indicator
8. Tap list item
9. Verify detail navigation
```

## Error State Testing

### Network Error Simulation
```bash
# Android - Toggle airplane mode
adb shell settings put global airplane_mode_on 1
adb shell am broadcast -a android.intent.action.AIRPLANE_MODE

# Restore
adb shell settings put global airplane_mode_on 0
```

### Test Scenarios
| Error Type | Trigger | Expected |
|------------|---------|----------|
| No network | Airplane mode | Error message + retry |
| Timeout | Slow network | Loading → timeout message |
| Server error | Invalid request | Error message |
| Empty data | No results | Empty state UI |

### Execution Pattern
```
1. Enable airplane mode
2. Trigger network action
3. Screenshot error state
4. Verify error message visible
5. Verify retry button present
6. Disable airplane mode
7. Tap retry
8. Verify recovery
```

## Accessibility Testing

### Checklist
- [ ] All interactive elements have labels
- [ ] Touch targets ≥ 44x44 points (iOS) / 48x48 dp (Android)
- [ ] Color contrast sufficient
- [ ] Text scalable
- [ ] Screen reader compatible

### Getting Accessibility Info
```bash
# Android - Check content descriptions
adb shell uiautomator dump
# Look for content-desc attributes

# iOS - Check accessibility labels
xcrun simctl ui booted describe window
```

### Common Issues
| Issue | Detection | Impact |
|-------|-----------|--------|
| Missing label | content-desc/label empty | Screen reader unusable |
| Small touch target | bounds < 44x44 | Hard to tap |
| Low contrast | Color analysis | Hard to read |
| Images without alt | No content-desc on ImageView | Info lost |

## Performance Indicators

### What to Monitor
- App launch time (< 2 seconds ideal)
- Screen transition time (< 300ms ideal)
- Scroll smoothness
- Memory usage growth
- Battery drain rate

### Launch Time Measurement
```bash
# Android
adb shell am start -W -n <package>/<activity>
# Look for "TotalTime" in output

# iOS (approximate via timing)
time xcrun simctl launch booted <bundle_id>
```

### Memory Monitoring
```bash
# Android
adb shell dumpsys meminfo <package>

# Watch for:
# - Heap size growing continuously
# - Native memory leaks
```

## Test Execution Priorities

### Critical Path (Always Test)
1. App launch without crash
2. Login/Authentication (if applicable)
3. Main feature access
4. Navigation between primary screens
5. Core business function

### Secondary (Time Permitting)
1. Settings/Preferences
2. Profile/Account management
3. Secondary features
4. Edge cases

### Regression Indicators
- Crash on any action
- Frozen/unresponsive UI
- Missing UI elements
- Broken navigation
- Data not saving

## Decision Tree for Autonomous Testing

```
START
  │
  ├─ Is there a login screen?
  │   ├─ YES → Test login flow first
  │   └─ NO → Proceed to main screen
  │
  ├─ Identify interactive elements
  │   ├─ Buttons → Queue for tapping
  │   ├─ Input fields → Queue for text input
  │   ├─ Lists → Queue for scroll + tap
  │   └─ Navigation → Queue for exploration
  │
  ├─ Execute queued actions
  │   ├─ Take screenshot before
  │   ├─ Perform action
  │   ├─ Wait for result
  │   ├─ Take screenshot after
  │   └─ Record result
  │
  ├─ Screen changed?
  │   ├─ YES → Add to visited screens, explore new screen
  │   └─ NO → Try next action
  │
  └─ All screens visited & actions tested?
      ├─ YES → Generate report
      └─ NO → Continue testing
```
