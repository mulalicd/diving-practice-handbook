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
