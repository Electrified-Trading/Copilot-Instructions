# 004 - Constants Organization Anti-Patterns

## ❌ **MISTAKES TO AVOID**

### 1. Separate Tooltip Constants
**ISSUE**: Creating individual constants for each tooltip creates maintenance burden.

```pinescript
// ❌ MAINTENANCE BURDEN
TOOLTIP_DISPLAY_MODE = "Choose visualization style..."
TOOLTIP_THRESHOLD = "Percentile threshold for heat detection..."
TOOLTIP_TRIANGLE_SCALE = "Size scale for triangle arrows..."

display_mode = input.string("Balanced", "Display Mode", tooltip=TOOLTIP_DISPLAY_MODE)
threshold = input.float(90.0, "Threshold", tooltip=TOOLTIP_THRESHOLD)
```

### 2. Inconsistent Naming Conventions
**ISSUE**: Mixed naming patterns make code harder to maintain.

```pinescript
// ❌ INCONSISTENT NAMING
MINIMAL_MODE = "Minimal"
MODE_BALANCED = "Balanced"
DISPLAY_DETAILED = "Detailed"
```

## ✅ **BETTER APPROACHES**

### 1. Inline Tooltips
**SOLUTION**: Define tooltips inline with input parameters (cleaner, fewer constants).

```pinescript
// ✅ CLEAN AND MAINTAINABLE
display_mode = input.string(MODE_TRIANGLE_ARROWS, DISPLAY_MODE_LABEL, 
  tooltip="Choose visualization style: Triangle Arrows (clean), Emoji + Percentage (detailed), or Off")

threshold = input.float(90.0, THRESHOLD_LABEL, 
  tooltip="Percentile threshold for showing heat signals")
```

### 2. Consistent Prefixed Naming
**SOLUTION**: Use consistent prefixes for logical grouping.

```pinescript
// ✅ CONSISTENT PREFIX PATTERN
MODE_MINIMAL = "Minimal" 
MODE_BALANCED = "Balanced"
MODE_DETAILED = "Detailed"

COLOR_HOT_100 = color.red
COLOR_HOT_75 = color.orange
COLOR_HOT_50 = color.yellow

LABEL_HOT = "HOT"
LABEL_COLD = "COLD"
LABEL_NEUTRAL = "NEUTRAL"
```

## 📋 **ORGANIZATION PRINCIPLES**

### Logical Grouping
Organize constants by function, not alphabetically:

```pinescript
///////////////////////////////////////////////////
// CONSTANTS
///////////////////////////////////////////////////

// UI Labels
INDICATOR_TITLE = "Statistical Price Heat Engine"
THRESHOLD_LABEL = "Threshold"
DISPLAY_MODE_LABEL = "Display Mode"

// Display Mode Options  
MODE_TRIANGLE_ARROWS = "Triangle Arrows"
MODE_EMOJI_PERCENTAGE = "Emoji + Percentage"
MODE_OFF = "Off"

// Color Constants
COLOR_HOT_100 = color.red
COLOR_COLD_100 = color.blue

// Alert Messages
EXTREME_HOT_MESSAGE = "Extreme overheated price movement detected"
EXTREME_COLD_MESSAGE = "Extreme overcooled price movement detected"
```

### When to Create Tooltip Constants
**CREATE** tooltip constants only when:
- Tooltip text is very long (>100 characters)
- Same tooltip used in multiple places
- Tooltip contains complex formatting

**DON'T CREATE** tooltip constants for:
- Simple explanations (<50 characters)
- One-time use tooltips
- Basic parameter descriptions

## 🎯 **LESSON LEARNED**

**Better constants organization reduces maintenance burden and improves code readability.**

### Best Practices
1. **Consistent naming**: Use prefixes like `MODE_`, `COLOR_`, `LABEL_`
2. **Logical grouping**: Group by function, not alphabetically
3. **Inline tooltips**: Prefer inline over separate constants
4. **Avoid redundancy**: Don't create constants for identical values

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Severity**: Medium - Code quality and maintainability
