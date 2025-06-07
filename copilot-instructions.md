# Pine Script Development Guidelines

## Purpose

This repository contains Pine Script indicators and utilities for advanced trend modeling and market behavior analysis on the TradingView platform. The code leverages modular design and innovative techniques like ensemble learning, adaptive resonance, and fractional memory.

---

## 🚨 Critical Pine Script v6 Constraints (Read First!)

### 1. Dynamic Series Call Restrictions
Pine Script runs bar-by-bar and **CANNOT** call functions like `ta.ema(...)`, `ta.atr(...)`, or `ta.sma(...)` with dynamic parameters (variable lengths) inside loops or conditionals.

**❌ This will fail:**
```pinescript
for i = 0 to states - 1
    ma_value = ta.ema(price, math.round(period * (1 + i * 0.2)))  // Dynamic length
```

**✅ Correct approach:**
```pinescript
// Precompute all variations outside loops
ema_state_0 = ta.ema(source, math.round(coherence_period * 1.0))
ema_state_1 = ta.ema(source, math.round(coherence_period * 1.2))
precomputed_emas = array.from(ema_state_0, ema_state_1, ...)

// Reference by index inside loops
for i = 0 to states - 1
    ma_value = array.get(precomputed_emas, i)
```

### 2. Type System Strictness
Pine Script v6 has stricter type requirements than v5. **Many functions now require constant values instead of series values.**

**Common patterns that fail in v6:**
- **Length parameters** (e.g., `ta.ema(...)`, `ta.sma(...)`, `ta.rsi(...)`) must be `simple int` (constant), not `series int`
- **Configuration parameters** (e.g., `plotshape(...)` size, `alertcondition(...)` message) must be constants, not dynamic
- **Function arguments** that were flexible in v5 may now require specific types
- **Mathematical operations** like `math.round()` return `series` but many functions expect `simple` types

**Solution patterns:**
```pinescript
// ❌ Will fail - series int to simple int
period_dynamic = math.round(input_period * volatility_factor)
ema_value = ta.ema(source, period_dynamic)

// ✅ Correct - use constant base with simple calculation
base_period = 20  // Simple int
period_multiplier = 1.5  // Simple float  
final_period = math.round(base_period * period_multiplier)  // Simple int
ema_value = ta.ema(source, final_period)
```

### 3. Function Declaration Order
Functions must be declared before they are used. **No forward references allowed.**

**Required Order:**
1. Version declaration and imports
2. Input parameters
3. **Utility functions** ← Must be here
4. Precomputed series values
5. Main calculations
6. Visualization and alerts

### 4. Missing Mathematical Functions
Pine Script doesn't include standard functions like `math.tanh`. Always check availability before use.

**Custom tanh implementation:**
```pinescript
tanh(series float x) =>
    if math.abs(x) > 10
        x > 0 ? 1.0 : -1.0
    else
        exp_2x = math.exp(2 * x)
        (exp_2x - 1) / (exp_2x + 1)
```

---

## 📋 Required Script Structure

### Version and Description
```pinescript
//@version=6
//@description <short summary of purpose>
```

### License Header
Use the Mozilla Public License 2.0 comment at the top of all files.

### Inputs Organization
- Group related `input.*` calls using `group="..."` 
- Keep related inputs on same line with `inline="..."`
- Use tooltips for non-obvious settings
- Avoid magic numbers—use named parameters

---

## 🎨 Visual Design Standards

### UI Guidelines
- Minimize clutter with checkboxes (`show_signals`, `show_states`, etc.)
- Gate all visuals behind user controls

### Color Scheme
- **Bullish**: lime or green
- **Bearish**: red  
- **Neutral/trend lines**: yellow or gray

### Performance Limits
- Avoid unnecessary `plotshape()` or `label.new()` calls
- Use `var` declarations for static/stateful arrays
- Cap loops with safe bounds (e.g., 3–12 for quantum states)

---

## 💻 Code Formatting Rules

### Multi-line Formatting
- Place operators (`and`, `or`, `+`, `-`, `?`, `:`) on following line
- **All continuations must use exactly 2 spaces indentation**
- Closing parentheses: same line OR separate line with 2 spaces

**Array.from() formatting:**
```pinescript
// ✅ Correct
precomputed_emas = array.from(
  ema_state_0, ema_state_1, ema_state_2,
  ema_state_3, ema_state_4, ema_state_5
  )  // Two-space indentation required

// ❌ Will be rejected
precomputed_emas = array.from(
  ema_state_0, ema_state_1, ema_state_2
)  // No indentation - compilation error
```

**Assignment patterns:**
```pinescript
variablea 
  = 1
  + anotherVariable

combined_signal 
 := quantum_trend * quantum_weight  // 1-space for 2-char oparators
  + source * art_weight             // 2-space for operators
```

### Naming Conventions
- Use descriptive, camelCase variables (`adaptiveTrend`, `kalmanGain`, `phaseDistance`)
- Separate sections with `///////////////////////////////////////////////////` dividers
- Comments explain "why", not "what"

---

## 🔧 Development Workflow

### Error Prevention Strategy
1. **Fix compilation errors first** - Syntax issues block everything
2. **Check constant vs. series requirements** - Many v6 functions need constants
3. **Verify indentation consistency** - Use regex search for spacing patterns
4. **Test alertcondition constraints** - String concatenation often fails
5. **Review plotshape parameters** - Size, color, etc. need constants

### When Refactoring
- Check language-specific indentation rules before reformatting
- Verify compilation after formatting changes
- Pay special attention to closing brackets, parentheses, and braces

---

## 🏗️ Architecture Guidelines

### Modularity
- For single-file testing: inline all logic
- For libraries: group logically (`QuantumState`, `KalmanFilter`, `FractionalMath`)
- Document all exported functions with parameters and return values
- Avoid mutable global state across modules

### Signal Logic
- Write signal conditions clearly in named variables
- Use `alertcondition()` with meaningful names and messages  
- Design for interpretability, not just accuracy

### When Uncertain
- Favor simplicity
- Annotate with `// TODO` or `// Heuristic`
- Provide multiple strategies if appropriate

---

## 📚 Quick Reference

### Common v6 Migration Issues
- `math.round()` returns `series int`, but `ta.ema(...)` needs `simple int`
- String concatenation in `alertcondition(...)` messages not allowed
- `ta.roc(...)` with length 0 causes runtime error
- Mixed indentation spacing causes compilation errors

### Precomputation Pattern
```pinescript
// Instead of dynamic calls in loops:
state_emas = array.from(
  ta.ema(source, 10), ta.ema(source, 15), ta.ema(source, 20),
  ta.ema(source, 25), ta.ema(source, 30), ta.ema(source, 35)
  )

for i = 0 to 5
    current_ema = array.get(state_emas, i)
    // Use current_ema in calculations
```

### Custom Function Template
```pinescript
custom_function(series float input, simple int param) =>
    // Implementation here
    result = input * param
    result
```
