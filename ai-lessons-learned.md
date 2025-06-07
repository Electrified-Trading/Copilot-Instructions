# AI Lessons Learned

This document captures important lessons learned during code development and refactoring that should be remembered for future iterations.

## Pine Script Formatting & Style

### Array.from() Formatting (June 6, 2025)

**Issue**: When making `array.from()` calls more vertical for readability, closing parentheses were placed on their own line without proper indentation.

**Incorrect Format**:
```pinescript
precomputed_emas = array.from(
  ema_state_0, ema_state_1, ema_state_2, ema_state_3,
  ema_state_4, ema_state_5, ema_state_6, ema_state_7,
  ema_state_8, ema_state_9, ema_state_10, ema_state_11
)  // ❌ No indentation
```

**Correct Format**:
```pinescript
precomputed_emas = array.from(
  ema_state_0, ema_state_1, ema_state_2, ema_state_3,
  ema_state_4, ema_state_5, ema_state_6, ema_state_7,
  ema_state_8, ema_state_9, ema_state_10, ema_state_11
  )  // ✅ Two-space indentation
```

**Rule**: In Pine Script multi-line continuations, closing parentheses must either:
1. End the same line as the last parameter, OR
2. Be on a separate line with exactly two spaces of indentation

**Reference**: From `copilot-instructions.md`: "All multi-line continuations should be indented with two spaces otherwise pine-script will reject them."

### Pine Script v6 Execution Constraints

**Issue**: Dynamic series calls (like `ta.ema()`, `ta.roc()`, `ta.atr()`) with variable parameters inside loops violate Pine Script execution constraints.

**Solution Pattern**:
1. **Precompute** all series values outside loops with constant parameters
2. Use **`array.from()`** for modern v6 array initialization instead of `var` + `array.set()`
3. **Reference precomputed values** by index inside loops instead of calculating dynamically

**Example**:
```pinescript
// ❌ Old approach - violates execution constraints
for i = 0 to states - 1
    ma_value = ta.ema(price, math.round(period * (1 + i * 0.2)))  // Dynamic call
    
// ✅ New approach - precompute and reference
ema_state_0 = ta.ema(source, math.round(coherence_period * 1.0))
ema_state_1 = ta.ema(source, math.round(coherence_period * 1.2))
// ... more precomputed values
precomputed_emas = array.from(ema_state_0, ema_state_1, ...)

for i = 0 to states - 1
    ma_value = i < array.size(precomputed_emas) ? array.get(precomputed_emas, i) : price
```

### Missing Pine Script Functions

**Issue**: Pine Script doesn't include all standard mathematical functions that developers might expect.

**Example**: `math.tanh` (hyperbolic tangent) doesn't exist in Pine Script and will cause compilation errors.

**Solution**: Implement custom functions using mathematical definitions:

```pinescript
// Custom tanh implementation with numerical stability
tanh(series float x) =>
    if math.abs(x) > 10
        x > 0 ? 1.0 : -1.0  // For large values, tanh approaches ±1
    else
        exp_2x = math.exp(2 * x)
        (exp_2x - 1) / (exp_2x + 1)
```

**Functions Confirmed Missing**: `math.tanh`

**Discovery Method**: Used `grep_search` to find all `math.tanh` usage, then replaced with custom implementation.

### Function Declaration Order

**Issue**: Pine Script requires functions to be declared before they are used. Unlike some languages, you cannot define functions after their invocation.

**Incorrect Structure**:
```pinescript
// ❌ This will cause compilation errors
confidence_multiplier = tanh(phase_strength / 100)  // Function used here

// Function defined later - TOO LATE!
tanh(series float x) =>
    // implementation
```

**Correct Structure**:
```pinescript
// ✅ Functions must be defined first
tanh(series float x) =>
    // implementation

// Then used later in calculations
confidence_multiplier = tanh(phase_strength / 100)
```

**Best Practice**: Place all utility functions immediately after input parameters and before any precomputed series or calculations.

**Recommended Order**:
1. Version declaration and imports
2. Input parameters
3. **Utility functions** ← Critical placement
4. Precomputed series values
5. Main calculation functions
6. Main calculation engine
7. Visualization and alerts

### Pine Script v6 Type System Strictness (June 6, 2025)

**Issue**: Pine Script v6 has stricter type requirements than v5, particularly around function parameters that must be constant vs. series values.

#### `ta.ema()` Length Parameter Constraint

**Problem**: `ta.ema(source, length)` requires `length` to be a `simple int` (constant), but `math.round()` of a series value produces a `series int`.

**Error**: `Cannot call "ta.ema" with argument "length"="call "math.round" (series int)". An argument of "series int" type was used but a "simple int" is expected.`

**Incorrect**:
```pinescript
adaptive_smoothing = adaptive_threshold(20, 14, 0.5) / sensitivity  // Returns series
adaptive_trend = ta.ema(combined_signal, math.round(adaptive_smoothing))  // ❌ series int
```

**Correct**:
```pinescript
adaptive_smoothing_base = 20  // Simple int constant
adaptive_smoothing_factor = math.max(0.5, math.min(2.0, 1.0 / sensitivity))  // Simple calculation
adaptive_smoothing_length = math.round(adaptive_smoothing_base * adaptive_smoothing_factor)  // Simple int
adaptive_trend = ta.ema(combined_signal, adaptive_smoothing_length)  // ✅ Simple int
```

#### `alertcondition()` Parameter Constraints

**Problem**: `alertcondition()` requires specific parameter types:
- Condition must be boolean but cannot use string variables in `ta.change()`
- Alert message must be constant string, not concatenated dynamic strings

**Incorrect**:
```pinescript
market_regime = "Trending"  // String variable
alertcondition(ta.change(market_regime) != 0, title, "Market regime changed to " + market_regime)  // ❌ Both issues
```

**Correct**:
```pinescript
regime_numerical = regime_score > 0.7 ? 1 : regime_score > 0.4 ? 0 : -1  // Numerical for ta.change()
alertcondition(ta.change(regime_numerical) != 0, title, "Market regime changed")  // ✅ Constant message
```

#### `plotshape()` Size Parameter Constraint

**Problem**: `plotshape()` `size` parameter must be a constant, not a dynamic series value.

**Incorrect**:
```pinescript
signal_size = confidence_multiplier > 0.7 ? size.normal : size.small  // Series value
plotshape(bullish_signal, size=signal_size)  // ❌ Series value not allowed
```

**Correct**:
```pinescript
plotshape(bullish_signal, size=size.normal)  // ✅ Constant value
```

**Root Cause**: Functions that accept configuration parameters (size, color, etc.) often require compile-time constants in v6 for performance optimization.

#### `ta.roc()` Length Parameter Constraint

**Problem**: `ta.roc(source, length)` requires `length` to be greater than 0. Using 0 causes a runtime error.

**Error**: `Invalid value of the 'length' argument (0) in the 'roc' function. It must be > 0.`

**Incorrect**:
```pinescript
roc_phase_0 = ta.roc(source, 0)  // ❌ Length cannot be 0
```

**Correct**:
```pinescript
roc_phase_0 = ta.roc(source, 1)  // ✅ Minimum valid length
```

**Note**: When designing phase space analysis or other mathematical algorithms that might calculate ROC with dynamic periods, always ensure the minimum length is 1.

#### Multiline Indentation Issues

**Problem**: Pine Script v6 is stricter about consistent indentation in multiline assignments.

**Incorrect**:
```pinescript
combined_signal 
                  := quantum_trend * quantum_weight  // ❌ Inconsistent spacing
    + source * art_weight                           // ❌ Mixed indentation
```

**Correct**:
```pinescript
combined_signal 
 := quantum_trend * quantum_weight  // ✅ Consistent 1-space for continuation
  + source * art_weight             // ✅ Consistent 2-space for operators
```

**Rule**: Pine Script v6 requires consistent indentation patterns. Use tools like `grep_search` with regex patterns to find inconsistent spacing.

### Pine Script v6 Migration Strategy

**Pattern Discovered**:
1. **Start with compilation errors** - Fix syntax issues first
2. **Check dynamic vs. constant requirements** - Many v6 functions need constants
3. **Verify indentation consistency** - Use regex search for spacing patterns
4. **Test alertcondition constraints** - String concatenation and series values often fail
5. **Review plotshape parameters** - Size, color, etc. often need constants

**Tools That Help**:
- `get_errors` tool to identify compilation issues
- `grep_search` with patterns like `\s{10,}` to find indentation inconsistencies
- `semantic_search` to find function usage patterns

**Key Insight**: Pine Script v6 prioritizes performance and compile-time optimization, which requires more explicit type distinctions between constants and series values.

## General Development Patterns

### When Making Code "More Vertical"

**Lesson**: When reformatting code for better readability by making it more vertical, always consider the language-specific indentation rules.

**Action Items**:
- Check the target language's style guide for multi-line formatting rules
- Verify compilation after formatting changes
- Pay special attention to closing brackets, parentheses, and braces

### Documentation Philosophy

**Pattern**: Maintain this lessons-learned document to capture:
1. Specific formatting issues encountered
2. Language-specific gotchas discovered
3. Best practices that emerge from real development scenarios
4. Cross-references to official style guides and documentation

This helps build institutional knowledge and prevents repeating the same mistakes.

---

*Last Updated: June 6, 2025*
*Context: Pine Script v6 Adaptive Quantum Trend indicator compilation fixes*
