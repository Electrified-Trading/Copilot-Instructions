# 001 - Incomplete Constants Migration

## ❌ **CRITICAL MISTAKE**

**ISSUE**: Failed to define all required color constants while migrating hardcoded colors.

### Missing Constants Found
- `COLOR_BG_HOT_EXTREME`, `COLOR_BG_COLD_EXTREME`, `COLOR_BG_HOT_VERY`, `COLOR_BG_COLD_VERY`
- `COLOR_WHITE`, `COLOR_BLACK`, `COLOR_GRAY`, `COLOR_TABLE_BG`

### Root Cause
When extracting constants, did not systematically verify ALL references were properly defined before replacing hardcoded values.

## ✅ **SOLUTION**

### Validation Process
Always grep search for the constants being used after defining them:

```bash
# Verify all color constants are defined
grep -n "COLOR_" file.pine | grep -v "="

# Find remaining hardcoded colors
grep 'color\.' file.pine | grep -v "COLOR_"
```

### Prevention Strategy
1. **Audit First**: Use grep to find ALL hardcoded references before starting
2. **Define All**: Create constants for every found reference
3. **Replace Systematically**: Go section by section
4. **Validate Frequently**: Check compilation after each section

## 🎯 **LESSON LEARNED**

**When extracting constants, must systematically verify ALL references are properly defined.**

Never assume you found all the references on the first pass. Always validate with automated searches.

---

**Project**: Statistical Price Heat Engine  
**Date**: June 7, 2025  
**Severity**: High - Caused compilation failures
