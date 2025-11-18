# MCP Server Migration Guide: Revamped UNITARES Framework

**Created:** November 16, 2025  
**Last Updated:** November 16, 2025  
**Status:** Migration Guide

## Overview

This guide provides step-by-step instructions for migrating the Governance MCP server from the current implementation to the revamped UNITARES framework with improved mathematics.

## Migration Summary

### Key Changes

1. **Void Integral (V)**: Pure integration → Leaky integrator (γ = 0.85)
2. **Integrity (I)**: Integrated computation → Direct computation (I = 1 - S)
3. **Energy (E)**: Differential equation → Direct normalized (optional)
4. **Lambda Control**: Add explicit clamping
5. **Void Detection**: Exact zero → Threshold-based

### Benefits

- ✅ Fixes accumulation weakness
- ✅ Prevents unbounded growth
- ✅ Guarantees bounded state variables
- ✅ Enables system recovery
- ✅ Simplifies implementation

---

## Pre-Migration Checklist

- [ ] Backup current MCP server code
- [ ] Backup existing CSV files
- [ ] Review current implementation
- [ ] Understand revamped framework
- [ ] Plan testing strategy
- [ ] Prepare rollback plan

---

## Step-by-Step Migration

### Step 1: Add New Parameters

**Add to MCP server configuration:**

```python
# New parameters for revamped framework
GAMMA = 0.85              # Void decay factor (leaky integrator)
VOID_THRESHOLD = 0.01     # Void detection threshold (instead of exact zero)
LAMBDA_MIN = 0.05         # Explicit minimum constraint
LAMBDA_MAX = 0.20         # Explicit maximum constraint
```

**Location:** MCP server config file or environment variables

---

### Step 2: Update Void Integral (V) Calculation

**Current Implementation:**
```python
# Pure integration
V_t = V_t_prev + (E_t - I_t)
```

**Revamped Implementation:**
```python
# Leaky integrator with decay
V_t = GAMMA * V_t_prev + (E_t - I_t)
# where GAMMA = 0.85
```

**Code Changes:**

```python
# Before
def calculate_void_integral(self, E, I, V_prev):
    """Calculate void integral using pure integration."""
    return V_prev + (E - I)

# After
def calculate_void_integral(self, E, I, V_prev, gamma=0.85):
    """Calculate void integral using leaky integrator."""
    return gamma * V_prev + (E - I)
```

**Migration Notes:**
- Store `V_t_prev` (previous V value) in state
- Initialize `V_t_prev = 0.0` for new agents
- Update `V_t_prev = V_t` after each calculation

---

### Step 3: Update Integrity (I) Calculation

**Current Implementation:**
```python
# Integrated computation
S_integral += S * dt
I = I0 - k * S_integral
```

**Revamped Implementation:**
```python
# Direct computation
I = 1.0 - S
```

**Code Changes:**

```python
# Before
def calculate_integrity(self, S, dt, I0=1.0, k=0.01):
    """Calculate integrity using integration."""
    self.S_integral += S * dt
    I = I0 - k * self.S_integral
    return I

# After
def calculate_integrity(self, S):
    """Calculate integrity directly from uncertainty."""
    # Ensure S is normalized [0, 1]
    S = max(0.0, min(1.0, S))
    I = 1.0 - S
    return I
```

**Migration Notes:**
- Remove `S_integral` state variable
- Remove `I0` and `k` parameters
- Ensure S is normalized before calculation
- I is now always bounded [0, 1]

**If Historical Memory Needed:**
```python
# Keep cumulative uncertainty as separate metric
self.S_cumulative += S * dt  # For historical tracking
I = 1.0 - S  # Current integrity (bounded)
```

---

### Step 4: Update Energy (E) Calculation (Optional)

**Current Implementation:**
```python
# Differential equation
dE/dt = alpha * (I - E)
```

**Revamped Implementation:**
```python
# Direct normalized computation
L_norm = min(token_len / max_token_norm, 1.0)
C_norm = min(latency / max_latency_norm, 1.0)
E = alpha_L * L_norm + alpha_C * C_norm
```

**Code Changes:**

```python
# Before (if using DE)
def calculate_energy(self, I, E_prev, alpha=0.5, dt=0.1):
    """Calculate energy using differential equation."""
    dE = alpha * (I - E_prev) * dt
    E = E_prev + dE
    return E

# After (direct normalized)
def calculate_energy(self, token_len, latency, 
                     max_token_norm=2048, max_latency_norm=5.0,
                     alpha_L=0.6, alpha_C=0.4):
    """Calculate energy directly from normalized inputs."""
    L_norm = min(token_len / max_token_norm, 1.0)
    C_norm = min(latency / max_latency_norm, 1.0)
    E = alpha_L * L_norm + alpha_C * C_norm
    return E
```

**Migration Notes:**
- Only change if currently using differential equation
- If E is already computed directly, no change needed
- Ensure inputs are normalized before calculation

---

### Step 5: Add Explicit Lambda Clamping

**Current Implementation:**
```python
# May not have explicit clamping
lambda_1 = lambda_1_prev + kp * e_V + ki * integral_V + k_rho * e_rho
```

**Revamped Implementation:**
```python
# Explicit clamping
lambda_uncapped = lambda_1_prev + kp * e_V + ki * integral_V + k_rho * e_rho
lambda_1 = max(LAMBDA_MIN, min(LAMBDA_MAX, lambda_uncapped))
```

**Code Changes:**

```python
# Before
def update_lambda(self, e_V, e_rho, lambda_prev, kp, ki, k_rho):
    """Update lambda without explicit clamping."""
    self.integral_V += e_V
    lambda_1 = lambda_prev + kp * e_V + ki * self.integral_V + k_rho * e_rho
    return lambda_1

# After
def update_lambda(self, e_V, e_rho, lambda_prev, kp, ki, k_rho,
                  lambda_min=0.05, lambda_max=0.20):
    """Update lambda with explicit clamping."""
    self.integral_V += e_V
    lambda_uncapped = (lambda_prev + 
                       kp * e_V + 
                       ki * self.integral_V + 
                       k_rho * e_rho)
    lambda_1 = max(lambda_min, min(lambda_max, lambda_uncapped))
    return lambda_1
```

**Migration Notes:**
- Add `lambda_min` and `lambda_max` parameters
- Ensure clamping is applied after all calculations
- Verify bounds are enforced

---

### Step 6: Update Void Detection

**Current Implementation:**
```python
# Exact zero detection
void_event = (V == 0.0)
```

**Revamped Implementation:**
```python
# Threshold-based detection
void_event = (abs(V) < VOID_THRESHOLD)
```

**Code Changes:**

```python
# Before
def detect_void(self, V):
    """Detect void event at exact zero."""
    return abs(V) < 1e-10  # Exact zero

# After
def detect_void(self, V, threshold=0.01):
    """Detect void event using threshold."""
    return abs(V) < threshold
```

**Migration Notes:**
- Set `VOID_THRESHOLD = 0.01` (configurable)
- More robust to numerical precision issues
- Catches near-void states

---

### Step 7: Update State Management

**State Variables to Add:**
```python
# New state variables
V_t_prev = 0.0  # Previous V for leaky integrator
```

**State Variables to Remove:**
```python
# Remove if using direct I
S_integral = 0.0  # No longer needed
I0 = 1.0          # No longer needed
```

**State Initialization:**
```python
# Updated initialization
def initialize_state(self):
    """Initialize revamped state variables."""
    self.E_t = 0.0
    self.S_t = 0.0
    self.I_t = 1.0  # Start with full integrity
    self.V_t = 0.0
    self.V_t_prev = 0.0  # NEW: For leaky integrator
    self.rho_t = 1.0
    self.lambda_1_t = 0.15
    self.integral_error_V = 0.0
    # Remove: self.S_integral = 0.0
```

---

### Step 8: Update CSV Format (If Needed)

**Current CSV Format:**
```csv
time,E,I,S,V,lambda1,coherence,void_event
```

**Revamped CSV Format:**
```csv
time,E,I,S,V,lambda1,coherence,void_event
# Same format, but values computed differently
```

**Migration Notes:**
- CSV format can stay the same
- Values will be computed using new formulas
- Existing CSV files remain compatible (read-only)

---

## Complete Code Example

### Before (Current Implementation)

```python
class GovernanceAgent:
    def __init__(self):
        self.E_t = 0.0
        self.S_t = 0.0
        self.I_t = 1.0
        self.V_t = 0.0
        self.S_integral = 0.0  # For integration
        self.rho_t = 1.0
        self.lambda_1_t = 0.15
        self.integral_error_V = 0.0
    
    def calculate_integrity(self, S, dt=0.1):
        """Current: Integrated computation."""
        self.S_integral += S * dt
        self.I_t = 1.0 - 0.01 * self.S_integral
        return self.I_t
    
    def calculate_void(self, E, I):
        """Current: Pure integration."""
        self.V_t = self.V_t + (E - I)
        return self.V_t
    
    def detect_void_event(self):
        """Current: Exact zero."""
        return abs(self.V_t) < 1e-10
    
    def update_lambda(self, e_V, e_rho):
        """Current: May not clamp."""
        self.integral_error_V += e_V
        self.lambda_1_t += (0.01 * e_V + 
                           0.001 * self.integral_error_V + 
                           0.005 * e_rho)
        return self.lambda_1_t
```

### After (Revamped Implementation)

```python
class GovernanceAgent:
    def __init__(self, gamma=0.85, void_threshold=0.01,
                 lambda_min=0.05, lambda_max=0.20):
        self.E_t = 0.0
        self.S_t = 0.0
        self.I_t = 1.0
        self.V_t = 0.0
        self.V_t_prev = 0.0  # NEW: For leaky integrator
        # REMOVED: self.S_integral
        self.rho_t = 1.0
        self.lambda_1_t = 0.15
        self.integral_error_V = 0.0
        
        # NEW: Parameters
        self.gamma = gamma
        self.void_threshold = void_threshold
        self.lambda_min = lambda_min
        self.lambda_max = lambda_max
    
    def calculate_integrity(self, S):
        """Revamped: Direct computation."""
        S = max(0.0, min(1.0, S))  # Normalize
        self.S_t = S
        self.I_t = 1.0 - S  # Direct, bounded
        return self.I_t
    
    def calculate_void(self, E, I):
        """Revamped: Leaky integrator."""
        self.V_t_prev = self.V_t  # Store previous
        self.V_t = self.gamma * self.V_t_prev + (E - I)
        return self.V_t
    
    def detect_void_event(self):
        """Revamped: Threshold-based."""
        return abs(self.V_t) < self.void_threshold
    
    def update_lambda(self, e_V, e_rho):
        """Revamped: Explicit clamping."""
        self.integral_error_V += e_V
        lambda_uncapped = (self.lambda_1_t + 
                          0.01 * e_V + 
                          0.001 * self.integral_error_V + 
                          0.005 * e_rho)
        self.lambda_1_t = max(self.lambda_min, 
                             min(self.lambda_max, 
                                 lambda_uncapped))
        return self.lambda_1_t
```

---

## Testing Procedure

### 1. Unit Tests

```python
def test_leaky_integrator():
    """Test leaky integrator bounds V."""
    agent = GovernanceAgent(gamma=0.85)
    
    # Simulate many updates
    for _ in range(100):
        agent.calculate_void(E=0.6, I=0.9)
    
    # V should be bounded
    assert abs(agent.V_t) < 10.0, "V should be bounded"
    print("✅ Leaky integrator test passed")

def test_integrity_bounds():
    """Test integrity stays bounded."""
    agent = GovernanceAgent()
    
    # Test with high uncertainty
    agent.calculate_integrity(S=2.0)  # Should clamp to 1.0
    assert 0.0 <= agent.I_t <= 1.0, "I should be bounded"
    
    # Test recovery
    agent.calculate_integrity(S=0.1)
    assert agent.I_t == 0.9, "I should recover immediately"
    print("✅ Integrity bounds test passed")

def test_lambda_clamping():
    """Test lambda stays in bounds."""
    agent = GovernanceAgent(lambda_min=0.05, lambda_max=0.20)
    
    # Try to push lambda out of bounds
    for _ in range(100):
        agent.update_lambda(e_V=100.0, e_rho=100.0)
    
    assert 0.05 <= agent.lambda_1_t <= 0.20, "Lambda should be clamped"
    print("✅ Lambda clamping test passed")
```

### 2. Integration Tests

```python
def test_with_actual_data():
    """Test with actual governance CSV data."""
    # Load existing CSV
    # Process updates with revamped framework
    # Compare results
    # Verify improvements
    pass
```

### 3. Regression Tests

- Verify existing functionality still works
- Check CSV compatibility
- Verify API compatibility
- Test agent state persistence

---

## Backward Compatibility

### CSV Files

- **Format:** Same CSV format (no changes needed)
- **Reading:** Existing CSV files can be read
- **Writing:** New CSV files use revamped calculations
- **Migration:** No migration needed for CSV files

### API Compatibility

- **Inputs:** Same input parameters
- **Outputs:** Same output format
- **Behavior:** Improved internal calculations
- **Breaking Changes:** None (internal improvements only)

### Agent State

- **New Agents:** Use revamped framework
- **Existing Agents:** Can continue with current or migrate
- **State Migration:** Optional (can start fresh)

---

## Rollback Plan

### If Issues Occur

1. **Revert Code:**
   ```bash
   git revert <migration-commit>
   ```

2. **Restore Backup:**
   ```bash
   cp backup/mcp_server.py mcp_server.py
   ```

3. **Restore State:**
   - CSV files remain compatible
   - No state migration needed

### Rollback Checklist

- [ ] Revert code changes
- [ ] Restore configuration
- [ ] Verify system works
- [ ] Check CSV files
- [ ] Monitor governance metrics

---

## Migration Checklist

### Pre-Migration
- [ ] Backup current code
- [ ] Backup CSV files
- [ ] Review revamped framework
- [ ] Plan testing strategy

### Migration Steps
- [ ] Add new parameters (gamma, void_threshold, etc.)
- [ ] Update V calculation (leaky integrator)
- [ ] Update I calculation (direct computation)
- [ ] Update E calculation (if needed)
- [ ] Add explicit lambda clamping
- [ ] Update void detection (threshold-based)
- [ ] Update state management
- [ ] Remove unused state variables

### Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Regression tests pass
- [ ] Test with actual data
- [ ] Verify CSV compatibility
- [ ] Verify API compatibility

### Post-Migration
- [ ] Monitor system behavior
- [ ] Verify improvements
- [ ] Update documentation
- [ ] Train team on changes

---

## Expected Improvements

### After Migration

1. **V Bounded:** V stays bounded (no unbounded growth)
2. **I Bounded:** I always [0, 1] (no negative values)
3. **Recovery:** System can recover from deviations
4. **Stability:** More stable mathematics
5. **Simplicity:** Simpler implementation

### Metrics to Monitor

- V value range (should be bounded)
- I value range (should be [0, 1])
- Void event frequency
- Lambda value range (should be [0.05, 0.20])
- System stability

---

## Troubleshooting

### Issue: V Still Growing Unbounded

**Solution:** Verify leaky integrator is implemented:
```python
V_t = gamma * V_t_prev + (E - I)  # Not: V_t = V_t + (E - I)
```

### Issue: I Going Negative

**Solution:** Verify direct computation:
```python
I = 1.0 - S  # Not: I = I0 - k * S_integral
```

### Issue: Lambda Out of Bounds

**Solution:** Verify explicit clamping:
```python
lambda_1 = max(lambda_min, min(lambda_max, lambda_uncapped))
```

---

## Support

For questions or issues during migration:
- Review: `docs/mcp/unitares-revamped-framework.md`
- Review: `docs/mcp/void-metric-analysis.md`
- Review: `docs/mcp/integrity-computation-comparison.md`

---

## Conclusion

This migration guide provides step-by-step instructions for updating the MCP server to use the revamped UNITARES framework. The changes improve mathematical stability while maintaining backward compatibility.

**Key Benefits:**
- ✅ Bounded state variables
- ✅ System recovery capability
- ✅ Simpler implementation
- ✅ Better stability

**Migration Time Estimate:** 2-4 hours for implementation + testing

---

**Status:** Ready for migration
