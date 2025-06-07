# 002 - Incorrect RGB Color Function Usage

## ❌ **CRITICAL MISTAKE**

**ISSUE**: Used nested `color.new()` functions instead of proper `color.rgb()` for RGB values.

### Wrong Implementation
```pinescript
// ❌ WRONG - Nested color.new() functions
COLOR_COLD_75 = color.new(color.new(0, 128, 255), 0)
```

### Root Cause
Confusion about Pine Script v6 color creation functions and their proper usage patterns.

## ✅ **CORRECT USAGE**

Pine Script v6 has specific functions for different color creation methods:

```pinescript
// ✅ CORRECT - Different methods for different purposes

// Create RGB color from values
COLOR_COLD_75 = color.rgb(0, 128, 255)

// Add transparency to existing color
COLOR_TRANSPARENT = color.new(color.red, 50)

// Create RGB with transparency in one call
COLOR_RGBA = color.rgb(255, 255, 0, 90)
```

## 📋 **FUNCTION REFERENCE**

| Function | Purpose | Example |
|----------|---------|---------|
| `color.rgb(r, g, b)` | Create RGB color | `color.rgb(0, 128, 255)` |
| `color.rgb(r, g, b, transparency)` | RGB with transparency | `color.rgb(255, 0, 0, 50)` |
| `color.new(base_color, transparency)` | Add transparency | `color.new(color.red, 50)` |

## 🚨 **NEVER DO THIS**

```pinescript
// ❌ NEVER nest color.new() functions
color.new(color.new(r, g, b), transparency)

// ❌ NEVER use color.new() for RGB creation
color.new(0, 128, 255)  // This is not valid syntax
```

## 🎯 **LESSON LEARNED**

**Always use the appropriate color function for the task:**
- `color.rgb()` for creating colors from RGB values
- `color.new()` for adding transparency to existing colors
- Never nest color functions

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Severity**: High - Caused compilation failures
