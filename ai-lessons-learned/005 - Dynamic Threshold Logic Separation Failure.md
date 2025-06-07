# 005 - Dynamic Threshold Logic Separation Failure

## ❌ **CRITICAL ARCHITECTURAL MISTAKE**

**ISSUE**: Mixed threshold logic causing unintended visual changes when user adjusts triangle threshold.

### The Problem
Used the same threshold calculations for both bar colors AND triangle logic:

```pinescript
// ❌ WRONG - Mixed threshold logic
base_threshold = math.max(threshold, triangle_threshold)  // This affects everything!

// Bar colors use this (WRONG!)
heat_color = if heat_percentile > base_threshold + 3.0
    COLOR_HOT_100
```

### User Experience Impact
- User changes triangle threshold from 95% to 97%
- **All bar colors shift** (should only affect triangles!)
- Background colors change (should only affect triangles!)
- Completely breaks user expectations

## ✅ **CORRECT SEPARATION**

### Separate Threshold Systems
Bar colors and triangle logic must use independent threshold calculations:

```pinescript
// ✅ CORRECT - Separate threshold systems

// BAR COLOR THRESHOLDS (only use main threshold)
bar_most_extreme_hot = threshold + 8.0      // e.g., 90% + 8% = 98%
bar_very_extreme_hot = threshold + 4.0      // e.g., 90% + 4% = 94%
bar_extreme_hot = threshold                 // e.g., 90%

// TRIANGLE THRESHOLDS (only use triangle_threshold)  
triangle_most_extreme_hot = triangle_threshold + 3.0  // e.g., 95% + 3% = 98%
triangle_very_extreme_hot = triangle_threshold + 1.5  // e.g., 95% + 1.5% = 96.5%
triangle_extreme_hot = triangle_threshold             // e.g., 95%

// Bar colors (never change when triangle_threshold changes)
heat_color = if heat_percentile > bar_most_extreme_hot
    COLOR_HOT_100
else if heat_percentile > bar_very_extreme_hot
    COLOR_HOT_75
else if heat_percentile > bar_extreme_hot
    COLOR_HOT_50
else
    na

// Triangle sizing (only changes when triangle_threshold changes)
triangle_size = if heat_percentile > triangle_most_extreme_hot
    base_size_most_extreme
else if heat_percentile > triangle_very_extreme_hot
    base_size_very_extreme
else
    base_size_extreme
```

## 📋 **DESIGN PRINCIPLES**

### Independent Control Systems
1. **Bar Colors**: Controlled by main `threshold` input
2. **Background Colors**: Controlled by main `threshold` input  
3. **Triangles**: Controlled by `triangle_threshold` input
4. **Alerts**: Controlled by main `threshold` input

### User Expectations
- **Main threshold** changes: Bar colors, background, alerts adjust
- **Triangle threshold** changes: ONLY triangles adjust (size, placement)
- **No cross-contamination** between the two systems

## 🚨 **COMMON MISTAKE PATTERNS**

### 1. Shared Base Calculation
```pinescript
// ❌ WRONG - Affects everything
base_threshold = math.max(threshold, triangle_threshold)
```

### 2. Mixed Variable Usage
```pinescript
// ❌ WRONG - Bar colors using triangle variables
heat_color = if heat_percentile > triangle_extreme_hot
```

### 3. Single Threshold System
```pinescript
// ❌ WRONG - Everything uses same logic
if heat_percentile > most_extreme_hot  // Used for bars AND triangles
```

## 🎯 **LESSON LEARNED**

**When designing multi-threshold systems, each control must have independent calculation logic.**

### Architecture Checklist
- [ ] Bar color thresholds calculated from main threshold only
- [ ] Triangle thresholds calculated from triangle_threshold only  
- [ ] Background thresholds calculated from main threshold only
- [ ] Alert thresholds calculated from main threshold only
- [ ] No shared variables between threshold systems
- [ ] User can adjust each threshold independently

### Testing Verification
1. Change main threshold → only bars/background/alerts change
2. Change triangle threshold → only triangles change
3. No unexpected visual shifts in unrelated elements

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Severity**: Critical - Breaks user experience
