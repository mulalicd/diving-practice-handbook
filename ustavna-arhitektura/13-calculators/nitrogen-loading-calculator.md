# Nitrogen Loading Calculator Specification

## Purpose
Calculate nitrogen tissue saturation and pressure groups for dive planning based on RDP methodology.

**Related Chapters:** Chapter 3 (Decompression Thinking), Chapter 6 (Nitrogen Loading), Chapter 12 (Dive Table Practice)

---

## Input Parameters

### Required Inputs:

**1. Depth** (meters or feet)
- Range: 5-40m (recreational limits)
- Validation: Must be positive integer
- Default: 18m

**2. Bottom Time** (minutes)
- Range: 1-220 minutes
- Validation: Must be positive integer
- Default: 30 minutes

**3. Starting Pressure Group** (for repetitive dives)
- Range: A-Z (26 letters)
- Default: A (first dive of day)
- Info tooltip: "Select group from previous dive"

### Optional Inputs:

**4. Altitude** (meters above sea level)
- Default: 0 (sea level)
- Range: 0-3000m
- Triggers altitude adjustment if > 300m
- Warning displayed if altitude diving

**5. Conservative Factor**
- Options: Standard / Conservative / Very Conservative
- Standard: Use exact RDP values
- Conservative: Reduce NDL by 10%
- Very Conservative: Reduce NDL by 20%
- Default: Standard

---

## Calculation Algorithm

### Step 1: Adjust for Altitude (if applicable)
if altitude > 300m:
theoretical_depth = actual_depth × (1 + altitude_adjustment_factor)
use theoretical_depth for all calculations

Altitude Adjustment Table:
- 300-600m: Use next deeper depth on tables
- 600-900m: Use 2 depths deeper
- 900-1200m: Use 3 depths deeper
- >1200m: Special altitude tables required

### Step 2: Determine Base Tissue Compartments

Use 5-compartment simplified model:
- **Fast tissue:** 5-minute half-time (muscles, blood)
- **Medium-fast:** 20-minute half-time (organs)
- **Medium:** 40-minute half-time (slower organs)
- **Medium-slow:** 80-minute half-time (bones, joints)
- **Slow tissue:** 120-minute half-time (fatty tissue)

### Step 3: Calculate Nitrogen Partial Pressure

For each compartment:
Ambient_Pressure_bar = (Depth_meters / 10) + 1
PN2_bar = 0.79 × Ambient_Pressure_bar

Example at 20m:
Ambient = (20/10) + 1 = 3 bar
PN2 = 0.79 × 3 = 2.37 bar

### Step 4: Calculate Tissue Loading

For each compartment:
Loading_Factor = 1 - e^(-ln(2) × Time / Half_Time)
Current_N2 = PN2 × Loading_Factor

Example: Fast tissue at 20m for 30 minutes:
Loading = 1 - e^(-0.693 × 30 / 5) = 0.985 (98.5% loaded)
Current_N2 = 2.37 × 0.985 = 2.33 bar

### Step 5: Determine Controlling Compartment
for each compartment:
compare Current_N2 to M-value (maximum allowable)
controlling_compartment = highest percentage of M-value

### Step 6: Match to RDP Pressure Group

Using controlling compartment:
- Compare nitrogen loading to RDP table
- Determine letter group A-Z
- Higher loading = letter further in alphabet

### Step 7: Calculate Remaining NDL
Table_NDL = lookup(depth, tables)
Remaining_NDL = Table_NDL - Bottom_Time
if Remaining_NDL < 0:
status = "Decompression Required"
elif Remaining_NDL < 5:
status = "Critical - Ascend Now"
elif Remaining_NDL < 10:
status = "Caution - Low Margin"
else:
status = "Safe"

---

## Output Format

### Primary Display:
┌─────────────────────────────────────────┐
│  NITROGEN LOADING RESULTS               │
├─────────────────────────────────────────┤
│                                         │
│  Dive Parameters:                       │
│  • Depth: 20m (3.0 bar)                │
│  • Bottom Time: 30 minutes             │
│  • Starting Group: A                    │
│  • Altitude: Sea Level                  │
│                                         │
│  Results:                               │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│  Ending Pressure Group: E               │
│  Remaining NDL: 15 minutes              │
│  Safety Status: ✓ GOOD                  │
│                                         │
│  Tissue Loading:                        │
│  Fast (5min):    ████████░░ 80%        │
│  Med-Fast (20):  ██████░░░░ 60%        │
│  Medium (40):    ████░░░░░░ 40%        │
│  Med-Slow (80):  ██░░░░░░░░ 20%        │
│  Slow (120):     █░░░░░░░░░ 10%        │
│                                         │
│  Critical Tissue: Fast (5min) - 80%    │
│  Recommended: Add safety margin         │
│                                         │
└─────────────────────────────────────────┘
[  Plan Repetitive Dive  ]  [  Export  ]

### Warning Display Examples:

**Yellow Warning (NDL < 10 min):**
⚠ WARNING: Only 8 minutes remaining at this depth
Recommendation: Begin ascent within 5 minutes

**Red Warning (NDL < 5 min):**
🔴 CRITICAL: Only 3 minutes to NDL!
ACTION REQUIRED: Ascend immediately to shallower depth

**Altitude Warning:**
⛰ ALTITUDE DIVING DETECTED (900m)
Tables adjusted to: 23m equivalent depth
Plan conservatively - decompression risk increased

---

## Validation Rules

### Input Validation:
```javascript
// Depth validation
if (depth < 5) {
  error("Minimum depth for calculation: 5 meters");
}
if (depth > 40) {
  error("Recreational limit exceeded. Maximum: 40 meters");
}

// Time validation
if (bottomTime < 1) {
  error("Bottom time must be at least 1 minute");
}
if (bottomTime > 220) {
  error("Time exceeds maximum table values");
}

// NDL check
maxNDL = getMaxNDL(depth);
if (bottomTime > maxNDL) {
  error(`Bottom time (${bottomTime}min) exceeds NDL (${maxNDL}min).
         Decompression diving requires technical training.`);
}
```

### Safety Validation:
```javascript
// Approaching limits
if (remainingNDL < 10 && remainingNDL >= 5) {
  showWarning("yellow", "Approaching NDL - plan ascent");
}

// Critical zone
if (remainingNDL < 5 && remainingNDL > 0) {
  showWarning("red", "CRITICAL - Ascend immediately");
}

// Reverse profile detection
if (isRepetitiveDive && currentDepth > previousDepth) {
  showWarning("orange", "Reverse profile detected - not recommended");
}

// Conservative factor reminder
if (conservativeFactor === "standard") {
  showInfo("Consider using conservative factor for added safety");
}
```

---

## User Interface Mockup
┌──────────────────────────────────────────────┐
│  🧮 Nitrogen Loading Calculator              │
├──────────────────────────────────────────────┤
│                                              │
│  📊 Dive Parameters                          │
│  ┌─────────────────────────────────────┐   │
│  │ Depth (meters)     [20] ▼     │   │
│  │                    [66] feet   │   │
│  │                                       │   │
│  │ Bottom Time (min)  [30]       │   │
│  │                                       │   │
│  │ Start Group        [A] ▼        │   │
│  │ ℹ First dive of day                  │   │
│  └─────────────────────────────────────┘   │
│                                              │
│  ⚙️ Advanced Options (click to expand)      │
│  ┌─────────────────────────────────────┐   │
│  │ ☐ Altitude Diving                    │   │
│  │   Altitude: [900] meters         │   │
│  │                                       │   │
│  │ Conservative Factor:                  │   │
│  │ ⚪ Standard ⚫ Conservative            │   │
│  │ ⚪ Very Conservative                   │   │
│  └─────────────────────────────────────┘   │
│                                              │
│     [     Calculate Nitrogen Loading     ]  │
│                                              │
├──────────────────────────────────────────────┤
│  Results appear here after calculation      │
│  (see Output Format section above)          │
└──────────────────────────────────────────────┘
Bottom Tabs:
[Formula View] [Tissue Graph] [RDP Table] [Export]

---

## Formula Reference Display

When user clicks "Formula View":
📐 Calculation Formulas

Ambient Pressure (bar):
P_ambient = (Depth_m / 10) + 1
Nitrogen Partial Pressure (bar):
PN2 = 0.79 × P_ambient
Tissue Loading (%):
Loading = 1 - e^(-ln(2) × t / t_half)
Current Nitrogen (bar):
N2_current = PN2 × Loading
Remaining NDL (minutes):
NDL_remaining = NDL_table - Time_bottom

Example Calculation:
Depth = 20m, Time = 30min, Tissue half-time = 40min
P_ambient = (20/10) + 1 = 3 bar
PN2 = 0.79 × 3 = 2.37 bar
Loading = 1 - e^(-0.693 × 30/40) = 0.598 = 59.8%
N2_current = 2.37 × 0.598 = 1.42 bar

---

## Tissue Graph Visualization

When user clicks "Tissue Graph":
- X-axis: Time (0 to bottom time)
- Y-axis: Nitrogen pressure (0 to max)
- 5 colored lines showing each compartment loading
- Horizontal line showing M-value limits
- Shaded region showing safe zone

---

## RDP Table Reference

Embedded mini-table for quick reference:

| Depth | Group A | Group C | Group E | Group G | NDL |
|-------|---------|---------|---------|---------|-----|
| 12m   | 10      | 20      | 30      | 40      | 147 |
| 15m   | 8       | 15      | 25      | 35      | 80  |
| 18m   | 6       | 12      | 20      | 30      | 56  |
| 20m   | 5       | 10      | 18      | 25      | 45  |
| 25m   | 4       | 8       | 13      | 18      | 29  |
| 30m   | 3       | 6       | 10      | 14      | 20  |

---

## Export Functionality

When user clicks "Export":

### PDF Export:
NITROGEN LOADING CALCULATION REPORT
Generated: [Date/Time]
Dive Parameters:

Depth: 20m
Time: 30 minutes
Starting Group: A
Altitude: Sea Level

Results:

Ending Group: E
Remaining NDL: 15 minutes
Safety Status: Good

Tissue Loading Details:
[Table with all 5 compartments]
Recommendations:

Suggested safety stop: 3-5 min at 5m
Suggested surface interval: 1:15 for Group C

DISCLAIMER: Educational calculation only...

### Copy to Clipboard:
Depth: 20m | Time: 30min | Group: A→E | NDL: 15min remaining

---

## Related Handbook Sections

**Essential Reading:**
- Chapter 3: Decompression Thinking
- Chapter 3: Dive Tables Explained
- Chapter 6: Nitrogen Loading
- Chapter 12: Dive Table Practice (Exercises 1-7)

**Link Display:**
Each section should be hyperlinked in the calculator interface.

---

## Safety Disclaimers

### Primary Disclaimer (always visible):
⚠️ EDUCATIONAL TOOL ONLY
This calculator is for educational purposes and dive planning
practice. Always use:

Certified dive computer for actual dives
Published dive tables as primary reference
Consult with dive professionals
Follow certification agency standards

Do NOT rely solely on this calculator for dive safety decisions.

### Before First Use (modal popup):
IMPORTANT SAFETY INFORMATION
Before using this calculator, you must understand:
✓ This is an educational simulation
✓ Real dive computers account for more factors
✓ Always maintain conservative safety margins
✓ Never exceed your certification limits
✓ Decompression diving requires technical training
[ ] I understand and accept these limitations
[Cancel]  [I Understand - Continue]

---

## Testing Requirements

### Validation Against Standards:
1. Cross-check all calculations against PADI RDP tables
2. Verify tissue compartment math matches Bühlmann model
3. Test edge cases (shallow depths, long times, altitude)
4. Confirm NDL calculations match published tables
5. Validate pressure group assignments

### User Testing:
1. Test with beginner divers (can they understand output?)
2. Test with instructors (is it pedagogically sound?)
3. Test on mobile devices (is UI responsive?)
4. Test accessibility (screen readers, color blindness)

### Error Testing:
1. Invalid inputs (negative numbers, letters, etc.)
2. Extreme values (0m, 100m, 999 minutes)
3. Impossible scenarios (Group Z at 10m for 5 min)

---

## Implementation Notes for Developers

### Performance:
- Calculations should complete in < 100ms
- Cache RDP table data locally
- Use Web Workers for complex calculations

### Accessibility:
- All controls keyboard-navigable
- ARIA labels on all inputs
- High contrast mode support
- Screen reader announcements for warnings

### Mobile Optimization:
- Touch-friendly input controls
- Larger tap targets (min 44x44px)
- Simplified layout for small screens
- Offline functionality (PWA)

---

**Specification Version:** 1.0  
**Last Updated:** February 2026  
**Status:** Ready for Development  
**Priority:** HIGH (Safety Critical)  
**Estimated Development Time:** 40-60 hours

Commit message: Calculator spec: Nitrogen loading with 5-compartment tissue model
Commit changes
