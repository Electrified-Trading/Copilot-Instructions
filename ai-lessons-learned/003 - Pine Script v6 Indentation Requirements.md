# 003 - Pine Script v6 Indentation Requirements

## ❌ **CRITICAL MISTAKE**

**ISSUE**: Mixed indentation styles causing compilation errors in Pine Script v6.

### Wrong Implementation
```pinescript
// ❌ WRONG - Inconsistent indentation
variableName 
= longExpression
+ anotherValue

// ❌ WRONG - No indentation on continuation
should_recalculate = bar_index != last_calculated_bar or 
math.abs(current_value - last_displacement_value) > 0.1
```

## ✅ **CORRECT IMPLEMENTATION**

Pine Script v6 requires **exactly 2 spaces** for continuation lines:

```pinescript
// ✅ CORRECT - Consistent 2-space indentation
variableName 
  = longExpression
  + anotherValue

// ✅ CORRECT - Operators at beginning of continuation lines
should_recalculate = bar_index != last_calculated_bar 
  or math.abs(current_value - last_displacement_value) > 0.1

// ✅ CORRECT - Multi-line function calls
label.new(
  bar_index, label_y, 
  text="", 
  style=triangle_style, 
  color=label_color, 
  size=triangle_size,
  tooltip=tooltip_text
  )
```

## 📋 **INDENTATION RULES**

### Continuation Lines
- **Exactly 2 spaces** for all continuation lines
- **Operators go at the beginning** of the continuation line (not end of previous)
- **Closing brackets** can be on same line or separate line with 2-space indent

### Array Formatting
```pinescript
// ✅ CORRECT
precomputed_emas = array.from(
  ema_state_0, ema_state_1, ema_state_2,
  ema_state_3, ema_state_4, ema_state_5
  )

// ❌ WRONG - No indentation
precomputed_emas = array.from(
ema_state_0, ema_state_1, ema_state_2
)
```

### Multi-line Conditionals
```pinescript
// ✅ CORRECT
heat_color = if heat_percentile > most_extreme_hot 
    or heat_percentile < most_extreme_cold
    COLOR_HOT_100
else if heat_percentile > very_extreme_hot 
    or heat_percentile < very_extreme_cold  
    COLOR_HOT_75
else
    na
```

## 🚨 **COMMON MISTAKES**

1. **Mixed spacing** - Some lines with 2 spaces, others with 4 or tabs
2. **Operators at line end** - Put `+`, `-`, `or`, `and` at beginning of next line
3. **No indentation** - Continuation lines flush with parent line
4. **Inconsistent bracket placement** - Be consistent throughout file

## 🎯 **LESSON LEARNED**

**Pine Script v6 indentation is strict and must be consistent throughout the entire file.**

Use regex searches to find inconsistent indentation:
```regex
^\s{1}[^\s]  # Lines with exactly 1 space (should be 2)
^\s{3}[^\s]  # Lines with 3 spaces (should be 2 or 4)
```

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Severity**: High - Prevents compilation
