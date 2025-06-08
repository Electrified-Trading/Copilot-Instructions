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
///
```

Note: placing `///` at the end of a function and an extra line after, helps editors to collapse it easily.

---

## 🚀 Advanced Pine Script Principles & Techniques

### 1. Pure Functions with Implicit Caching
Prefer pure functions that return `series` outputs. Pine Script's implicit caching means calling a function with the same parameter values across bars creates a cached series that Pine can reference efficiently using back-referencing.

**Example:**
```pinescript
// Pure function - Pine Script caches results automatically
get_adaptive_ma(series float source, series int length, series float factor) =>
    alpha = 2.0 / (length + 1) * factor
    var float ema_value = na
    ema_value := na(ema_value) ? source : alpha * source + (1 - alpha) * ema_value
    ema_value

// Each unique parameter combination gets its own cached series
fast_ma = get_adaptive_ma(close, 10, 1.5)
slow_ma = get_adaptive_ma(close, 20, 1.0)

// Back-reference cached series - Pine Script efficiently retrieves historical values
ma_crossover = ta.crossover(fast_ma, slow_ma)
ma_divergence = fast_ma - fast_ma[5]  // Previous values accessible via caching
trend_strength = (fast_ma - slow_ma) / slow_ma[10] * 100  // Historical comparison

// Multiple calls with same parameters reuse cached series (no recalculation)
ma_signal = get_adaptive_ma(close, 10, 1.5)  // Reuses fast_ma cache
previous_signal = ma_signal[3]  // Back-reference works seamlessly
```

### 2. Bar-Scoped Memory with Parameterized Functions
Use `var` inside functions to persist state across bars. When combined with parameterized function calls, this enables bar-scoped memory where each unique argument set gets its own persistent storage.

**Example:**
```pinescript
// Function maintains separate state for each parameter combination
rolling_zscore(series float value, series int window) =>
    var array<float> data = array.new<float>()
    
    // Each unique window size gets its own array
    array.push(data, value)
    if array.size(data) > window
        array.shift(data)
    
    if array.size(data) >= window
        mean = array.avg(data)
        std_dev = array.stdev(data)
        std_dev > 0 ? (value - mean) / std_dev : 0.0
    else
        na

// Different window sizes maintain separate internal arrays
zscore_short = rolling_zscore(close, 20)
zscore_long = rolling_zscore(close, 50)
```

### 3. Encapsulated Statistical Calculations
Functions like `get_zscore(windowSize, displacement, isPositive)` can encapsulate statistical calculations. Internally, these functions may maintain rolling arrays of historical values and compute results like mean, standard deviation, or z-score dynamically.

**Example:**
```pinescript
// Encapsulated statistical engine with internal state management
get_directional_zscore(series float displacement, series int window, series bool is_positive) =>
    var array<float> pos_history = array.new<float>()
    var array<float> neg_history = array.new<float>()
    
    // Separate positive and negative samples
    if displacement > 0 and is_positive
        array.push(pos_history, displacement)
        if array.size(pos_history) > window
            array.shift(pos_history)
    else if displacement < 0 and not is_positive
        array.push(neg_history, math.abs(displacement))
        if array.size(neg_history) > window
            array.shift(neg_history)
    
    // Calculate z-score using appropriate distribution
    relevant_data = is_positive ? pos_history : neg_history
    if array.size(relevant_data) >= 10
        mean = array.avg(relevant_data)
        std_dev = array.stdev(relevant_data)
        std_dev > 0 ? (math.abs(displacement) - mean) / std_dev : 0.0
    else
        na
```

### 4. Modular Statistical Structures with Records
To represent arrays of arrays or modular statistical structures, wrap arrays in records or objects. These can be stored in a master array or dictionary keyed by parameters, supporting scalable and layered models while maintaining a functional structure.

**Example:**
```pinescript
// Statistical module as a record/type
type StatEngine
    array<float> data
    float mean
    float std_dev
    int max_size

// Factory function for creating statistical engines
new_stat_engine(simple int max_samples) =>
    StatEngine.new(
        data = array.new<float>(),
        mean = na,
        std_dev = na,
        max_size = max_samples
    )

// Method to update statistical engine
update_engine(StatEngine engine, series float value) =>
    array.push(engine.data, value)
    if array.size(engine.data) > engine.max_size
        array.shift(engine.data)
    
    if array.size(engine.data) >= 10
        engine.mean := array.avg(engine.data)
        engine.std_dev := array.stdev(engine.data)
    
    engine

// Multi-scale statistical system using records
var StatEngine short_engine = new_stat_engine(20)
var StatEngine medium_engine = new_stat_engine(50)
var StatEngine long_engine = new_stat_engine(100)

// Update all engines with current displacement
current_displacement = (close - close[20]) / close[20] * 100
short_engine := update_engine(short_engine, current_displacement)
medium_engine := update_engine(medium_engine, current_displacement)
long_engine := update_engine(long_engine, current_displacement)
```

### 5. Best Practices for SPHE and Advanced Indicators

**State Management:**
- Use parameterized functions with internal `var` declarations for component isolation
- Leverage Pine Script's automatic caching for performance optimization
- Maintain separate statistical distributions for different market conditions

**Scalability:**
- Wrap complex data structures in records/types for modularity
- Create factory functions for consistent initialization
- Use method-like functions for state updates and calculations

**Performance:**
- Minimize global variables by encapsulating state in functions
- Take advantage of Pine Script's series caching for repeated calculations
- Use records to group related data and reduce parameter passing

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
- Prefer using CONSTANT = "Constant" at the top of the file for reused strings
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
