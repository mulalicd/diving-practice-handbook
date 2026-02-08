# Calculator Specifications

## Purpose

Technical specifications for interactive calculators that enable divers to perform safety-critical calculations. These specs guide web app development.

**Related Chapters:** Chapter 6 (Gas & Physiology), Chapter 3 (Forgotten Foundations)

---

## Calculator List

### 1. Nitrogen Loading Calculator
**File:** `nitrogen-loading-calculator.md`  
**Purpose:** Calculate tissue nitrogen saturation and pressure groups

### 2. Oxygen Toxicity Calculator  
**File:** `oxygen-toxicity-calculator.md`  
**Purpose:** Calculate PPO2 and CNS oxygen exposure tracking

### 3. Decompression Planner
**File:** `decompression-planner.md`  
**Purpose:** Plan dive profiles with NDL and deco stop calculations

### 4. Gas Mixture Analyzer
**File:** `gas-mixture-analyzer.md`  
**Purpose:** Analyze gas blends and calculate MOD

### 5. Air Consumption Calculator
**File:** `air-consumption-calculator.md`  
**Purpose:** Calculate SAC rate and plan gas requirements

---

## Implementation Requirements

**All calculators must:**
- Provide step-by-step calculation display
- Include input validation (realistic ranges)
- Show formulas used
- Display safety warnings when limits approached
- Support metric and imperial units
- Export results for dive planning
- Include educational tooltips

**Safety Critical:**
- Always include disclaimers
- Flag dangerous values in red
- Require confirmation for edge cases
- Link to relevant handbook chapters

---

## Web App Integration

Calculators should be:
- Accessible from relevant handbook chapters
- Available in standalone calculator section
- Mobile-responsive design
- Offline-capable (Progressive Web App)

---

**Document Version:** 1.0  
**Last Updated:** February 2026  
**Status:** Specification Complete
