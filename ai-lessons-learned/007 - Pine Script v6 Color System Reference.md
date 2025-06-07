# 007 - Pine Script v6 Color System Reference

## 📋 **COMPLETE COLOR FUNCTION REFERENCE**

### Core Color Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `color.rgb(r, g, b)` | Create RGB color | `color.rgb(0, 128, 255)` |
| `color.rgb(r, g, b, transparency)` | RGB with transparency | `color.rgb(255, 0, 0, 50)` |
| `color.new(base_color, transparency)` | Add transparency | `color.new(color.red, 50)` |

### Built-in Colors
- `color.red`, `color.blue`, `color.green`
- `color.yellow`, `color.orange`, `color.purple`
- `color.white`, `color.black`, `color.gray`
- `color.aqua`, `color.lime`, `color.navy`

## ✅ **CORRECT USAGE PATTERNS**

### RGB Color Creation
```pinescript
// ✅ CORRECT - Create RGB colors
COLOR_BLUE_CYAN = color.rgb(0, 128, 255)      // Blue-cyan midpoint
COLOR_CUSTOM_RED = color.rgb(220, 20, 60)     // Crimson
COLOR_LIME_GREEN = color.rgb(50, 205, 50)     // Lime green

// ✅ CORRECT - RGB with transparency
COLOR_SEMI_TRANSPARENT = color.rgb(255, 255, 0, 50)  // 50% transparent yellow
```

### Adding Transparency
```pinescript
// ✅ CORRECT - Add transparency to existing colors
COLOR_TRANSPARENT_RED = color.new(color.red, 75)      // 75% transparent red
COLOR_SUBTLE_BLUE = color.new(color.blue, 90)         // 90% transparent blue (very subtle)

// ✅ CORRECT - Transparency to custom colors
COLOR_BASE = color.rgb(0, 128, 255)
COLOR_BASE_TRANSPARENT = color.new(COLOR_BASE, 60)
```

### Background Colors
```pinescript
// ✅ CORRECT - Subtle background highlighting
COLOR_BG_HOT_EXTREME = color.new(color.red, 95)       // 95% transparent = very subtle
COLOR_BG_HOT_VERY = color.new(color.orange, 95)
COLOR_BG_COLD_EXTREME = color.new(color.blue, 92)
COLOR_BG_COLD_VERY = color.new(color.blue, 94)
```

## ❌ **COMMON MISTAKES**

### Never Nest Color Functions
```pinescript
// ❌ WRONG - Never do this
color.new(color.new(0, 128, 255), 0)
color.new(color.rgb(255, 0, 0), 50)  // Redundant

// ✅ CORRECT - Use appropriate function
color.rgb(0, 128, 255)                // For RGB
color.rgb(255, 0, 0, 50)             // For RGB with transparency
```

### Invalid RGB Usage
```pinescript
// ❌ WRONG - color.new() is not for RGB creation
COLOR_CUSTOM = color.new(0, 128, 255)  // Invalid syntax

// ✅ CORRECT - Use color.rgb() for RGB values
COLOR_CUSTOM = color.rgb(0, 128, 255)
```

### Transparency Confusion
```pinescript
// ❌ WRONG - Adding transparency to already transparent color
base_transparent = color.new(color.red, 50)
more_transparent = color.new(base_transparent, 75)  // Confusing

// ✅ CORRECT - Set final transparency directly
COLOR_FINAL = color.new(color.red, 75)
```

## 🎨 **COLOR SCHEME DESIGN**

### Progressive Color Systems
Design colors that work together visually:

```pinescript
// Hot progression (decreasing intensity)
COLOR_HOT_100 = color.red        // Most extreme (95%+)
COLOR_HOT_75 = color.orange      // Very hot (90-95%)
COLOR_HOT_50 = color.yellow      // Hot (80-90%)

// Cold progression (decreasing intensity)
COLOR_COLD_100 = color.blue      // Most extreme (<5%)
COLOR_COLD_75 = color.rgb(0, 128, 255)  // Very cold (5-10%) - Mathematical midpoint
COLOR_COLD_50 = color.aqua       // Cold (10-20%)
```

### Mathematical Color Calculations
For midpoint colors between two extremes:
```pinescript
// Blue to Cyan midpoint calculation:
// Blue = RGB(0, 0, 255)
// Cyan = RGB(0, 255, 255)  
// Midpoint = RGB(0, 128, 255)
COLOR_BLUE_CYAN_MID = color.rgb(0, 128, 255)
```

## 🔧 **VALIDATION TECHNIQUES**

### Find Hardcoded Colors
```bash
# Find all color. references that aren't constants
grep 'color\.' file.pine | grep -v "COLOR_"

# Find any remaining RGB values  
grep -E 'rgb\([0-9]' file.pine | grep -v "COLOR_"
```

### Test Color Visibility
- **Dark themes**: Test with dark TradingView theme
- **Light themes**: Test with light TradingView theme  
- **Transparency levels**: 90-95% for subtle backgrounds, 0-50% for visible elements
- **Color blindness**: Test red/green combinations

## 🎯 **BEST PRACTICES**

1. **Use descriptive names**: `COLOR_HOT_100` not `RED_COLOR`
2. **Group by intensity**: `COLOR_HOT_100`, `COLOR_HOT_75`, `COLOR_HOT_50`
3. **Document purpose**: Comment what percentile ranges colors represent
4. **Mathematical precision**: Use calculated midpoints for smooth progressions
5. **Test visibility**: Verify colors work on both light and dark themes

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Category**: Technical Reference
