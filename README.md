# 🌞 Ok-Sizing Tool 
This web-based calculator helps estimate **energy, battery, and solar panel requirements** for small embedded or IoT systems such as sensor nodes, environmental monitors, or smart farming devices.

It computes:
- Average load (with and without sleep mode)
- Daily energy consumption
- Battery capacity and configuration
- Suitable BMS and charge controller
- Recommended solar panel wattage (including 2× oversizing margin)

---

## 🚀 Features

✅ **Load Estimation**  
Add multiple components, specifying voltage and current draw for both sleep and active states.  
Uses duty cycle timing to compute realistic average power consumption.

✅ **Sleep Mode vs No-Sleep Mode**  
Compares power usage and shows savings from low-power operation.

✅ **Battery Sizing**  
Calculates required battery capacity (Ah and Wh) for a given autonomy period.  
Includes battery efficiency and depth-of-discharge adjustments.

✅ **Battery Pack Configuration**  
Suggests the number of cells in series and parallel based on chemistry (LiFePO₄, Li-ion, or Lead-acid).  
Outputs total pack voltage and energy capacity.

✅ **Recommended BMS & Charge Controller**  
Suggests appropriate BMS type (e.g., 4S LiFePO₄) and current rating.  
Suggests whether to use PWM or MPPT controller, with proper current rating.

✅ **Solar Panel Recommendation**  
Computes required panel wattage and doubles it (2× rule) to cover inefficiencies, cloudy days, and aging.

---

## 🧮 How It Works

1. **Loads / Components**
   - Enter each component’s operating voltage, current in sleep, and current when active.
   - You can add or remove rows as needed.

2. **Duty Cycle / System Settings**
   - Define how long the system stays awake and asleep per cycle (in minutes).  
   - Example: 10 min cycle, 2 min awake, 8 min sleep.

3. **Battery Settings**
   - Choose your chemistry:
     - LiFePO₄ (3.2 V per cell)
     - Li-ion (3.7 V per cell)
     - Lead-acid (2.0 V per cell)
   - Specify single-cell capacity (Ah).  
   - Enter autonomy days, depth of discharge, battery efficiency, and sun hours per day.

4. **Click “Calculate”**
   - The tool automatically computes all results and displays them in clear sections.

---
## 🔍 How the Math Works (Step by Step)

This section shows exactly how each number is calculated.  
We’ll use these example results:

- Average Load (Sleep Mode): 0.613 Wh/hr → 14.72 Wh/day  
- Average Load (No-Sleep Mode): 2.670 Wh/hr → 64.08 Wh/day  
- Battery Requirement (Sleep Mode): 4.84 Ah @ 12 V → 58.1 Wh stored  
- Battery Pack Configuration: 4 cells in series × 2 parallel (LiFePO₄ 3.2 V, 3.2 Ah each)

### 1. Per-component power
For each component you enter:
- Voltage = `V`
- Sleep current = `I_sleep` (mA)
- Awake current = `I_awake` (mA)

Power in each state:
- Awake power (W):  
  \[
  P_\text{awake} = V \times \frac{I_\text{awake}}{1000}
  \]
- Sleep power (W):  
  \[
  P_\text{sleep} = V \times \frac{I_\text{sleep}}{1000}
  \]

We do this for every row (Pi, ESP32, sensors, etc.) and sum them.

### 2. Duty cycle weighting
You tell the tool:
- `Awake per cycle (min)` = \( t_\text{awake} \)
- `Sleep per cycle (min)` = \( t_\text{sleep} \)
- `Cycle length (min)` = \( t_\text{awake} + t_\text{sleep} \)

We convert those to fractions of time:
\[
f_\text{awake} = \frac{t_\text{awake}}{t_\text{awake} + t_\text{sleep}}
\]
\[
f_\text{sleep} = \frac{t_\text{sleep}}{t_\text{awake} + t_\text{sleep}}
\]

Example (10 min cycle, 2 min awake, 8 min sleep):
\[
f_\text{awake} = \frac{2}{10} = 0.2
\quad,\quad
f_\text{sleep} = \frac{8}{10} = 0.8
\]

Now we compute the **time-weighted average power** of each component:
\[
P_\text{avg,component} = P_\text{awake} \times f_\text{awake} \;+\; P_\text{sleep} \times f_\text{sleep}
\]

Then we add all components to get **system average power**:
\[
P_\text{avg,total} = \sum P_\text{avg,component}
\]

That total is what we report as:
> **Average Load (Sleep Mode)** in Wh/hr  
(it’s effectively Watts, i.e. Watt-hours per hour)

In your example:
\[
P_\text{avg,total} = 0.613 \text{ Wh/hr}
\]

#### No-Sleep Mode
For “No-Sleep Mode”, we assume every component is awake 100% of the time:
\[
P_\text{nosleep,total} = \sum \left(V \times \frac{I_\text{awake}}{1000}\right)
\]
That gave:
\[
P_\text{nosleep,total} = 2.670 \text{ Wh/hr}
\]

### 3. Daily energy use (Wh/day)
We multiply by 24 hours:
\[
\text{Wh/day} = \text{(Wh/hr)} \times 24
\]

So:
\[
0.613 \times 24 = 14.72 \text{ Wh/day}
\]
\[
2.670 \times 24 = 64.08 \text{ Wh/day}
\]

### 4. Energy savings from sleep mode
We show how much energy you save vs running everything awake 24/7:
\[
\text{Savings \%} =
\left(
1 - \frac{\text{Sleep Wh/day}}{\text{No-Sleep Wh/day}}
\right) \times 100
\]

Plugging in:
\[
\left(
1 - \frac{14.72}{64.08}
\right)\times 100
= 77.0\%
\]

So sleep mode cuts your daily energy by ~77%.

### 5. Battery capacity required (Ah)
You tell the tool:
- Autonomy (days with no sun) = \( D \)
- Depth of Discharge (DoD, e.g. 80% = 0.8)
- Battery efficiency (e.g. 95% = 0.95)
- Battery bus voltage \( V_\text{bus} \) (e.g. 12 V or 12.8 V)

First we find total watt-hours you need to survive:
\[
E_\text{total} = \text{Wh/day} \times D
\]

Then we account for usable fraction of the battery:
\[
\text{usable fraction} = \text{DoD} \times \text{efficiency}
\]

Then convert required watt-hours into required amp-hours:
\[
\text{Battery Ah} =
\frac{E_\text{total}}{V_\text{bus} \times \text{usable fraction}}
\]

Using your numbers:
- Daily Wh/day (Sleep Mode) = 14.72
- Autonomy = 2 days (example)
- DoD = 80% = 0.8
- Efficiency = 95% = 0.95
- Bus voltage = 12 V (≈ 12.8 V nominal 4S LiFePO₄)

Step 1:
\[
E_\text{total} = 14.72 \times 2 = 29.44 \text{ Wh}
\]

Step 2:
\[
\text{usable fraction} = 0.8 \times 0.95 = 0.76
\]

Step 3:
\[
\text{Battery Ah} =
\frac{29.44}{12 \times 0.76}
\approx 4.84 \text{ Ah}
\]

That becomes:
> `Battery Requirement (Sleep Mode): 4.84 Ah @ 12 V → 58.1 Wh stored`

Note: “58.1 Wh stored” is just Ah × Vbus:
\[
4.84 \text{ Ah} \times 12 \text{ V} \approx 58.1 \text{ Wh}
\]

### 6. Battery pack configuration (series × parallel)
You also enter:
- Cell chemistry voltage (LiFePO₄ = 3.2 V/cell)
- Cell capacity (e.g. 3.2 Ah per cell)

**Series cells** needed:
We figure out how many cells you need in series to reach your bus voltage:
\[
N_\text{series} = \left\lceil \frac{V_\text{bus}}{V_\text{cell}} \right\rceil
\]

For LiFePO₄:
- Cell ≈ 3.2 V
- Bus ≈ 12.8 V
\[
N_\text{series} = \lceil 12.8 / 3.2 \rceil = \lceil 4 \rceil = 4
\]
That gives you a 4S pack (~12.8 V nominal).

**Parallel strings** needed:
We check how many in parallel are needed to meet the Ah requirement:
\[
N_\text{parallel} = \left\lceil \frac{\text{Required Ah}}{\text{Cell Ah}} \right\rceil
\]

Example:
- Required Ah: 4.84 Ah
- Each cell: 3.2 Ah
\[
N_\text{parallel} = \lceil 4.84 / 3.2 \rceil = \lceil 1.51 \rceil = 2
\]

So you need **2 parallel strings** of those 4 cells.

Final pack:
\[
4\text{S} \times 2\text{P} = 8 \text{ cells total}
\]

We present that as:
> `4 cells in series × 2 parallel = 8 cells total (3.2 V / 3.2 Ah each)`

This also tells you what BMS you need:
- Because it’s 4 cells in series → you need a **4S LiFePO₄ BMS**
- Because you have 2P → the current capability is higher, so pick a BMS with enough continuous current rating

### 7. Solar panel sizing
We know the daily energy the node needs (Wh/day).  
We also know:
- "System Eff. PV→load (%)": efficiency from sunlight through charge controller into battery and into load.
- "Sun Hours / day": effective full-sun hours (e.g. 7)

We compute how many watt-hours the panel must deliver per day:
\[
\text{Panel Wh/day} =
\frac{\text{Load Wh/day}}{\text{System Efficiency}}
\]

Then turn that into panel watts:
\[
\text{Panel W}_\text{ideal} =
\frac{\text{Panel Wh/day}}{\text{Sun Hours/day}}
\]

Then we double it (2× rule) for safety:
\[
\text{Panel W}_\text{recommended} = 2 \times \text{Panel W}_\text{ideal}
\]

So the tool prints both:
- the calculated panel size
- the recommended “2×” panel size you should actually buy

---

## 🧑‍🔧 Author Notes

This tool was developed to support low-power embedded systems and smart sensor deployments (IoT nodes, weather/rain sensors, agricultural monitoring, etc.). It’s suitable for academic use, prototyping, and real field deployments.

**File name:** `index.html`  
**Tech:** Plain HTML + CSS + JavaScript (no backend)  
**License:** MIT  
**Version:** 1.0  
**Date:** 2025
