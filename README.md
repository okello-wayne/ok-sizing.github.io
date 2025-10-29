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

## 🔍 How the Math Works (Step by Step)

This section shows exactly how each number is calculated, using these example results:

- Average Load (Sleep Mode): 0.613 Wh/hr → 14.72 Wh/day  
- Average Load (No-Sleep Mode): 2.670 Wh/hr → 64.08 Wh/day  
- Battery Requirement (Sleep Mode): 4.84 Ah @ 12 V → 58.1 Wh stored  
- Battery Pack Configuration: 4 cells in series × 2 parallel (LiFePO₄ 3.2 V / 3.2 Ah each)

---

### 1. Per-component power

For each component:

P_awake = V × (I_awake / 1000)
P_sleep = V × (I_sleep / 1000)

yaml
Copy code

Power is in watts. All components’ awake and sleep powers are summed.

---

### 2. Duty cycle weighting

If your cycle is 10 min total (2 min awake, 8 min sleep):

f_awake = 2 / 10 = 0.2
f_sleep = 8 / 10 = 0.8

perl
Copy code

Average power for each component:

P_avg(component) = P_awake × f_awake + P_sleep × f_sleep

perl
Copy code

Then total system power:

P_avg(total) = sum of all P_avg(component)

mathematica
Copy code

That’s reported as **Average Load (Sleep Mode)**.  
For “No-Sleep Mode,” everything is assumed awake:

P_nosleep = Σ(V × I_awake / 1000)

yaml
Copy code

---

### 3. Daily energy use (Wh/day)

Multiply average power by 24 hours:

Wh/day = (Wh/hr) × 24

makefile
Copy code

Example:

0.613 × 24 = 14.72 Wh/day
2.670 × 24 = 64.08 Wh/day

yaml
Copy code

---

### 4. Energy savings from sleep mode

Savings % = (1 − (Sleep Wh/day ÷ No-Sleep Wh/day)) × 100

makefile
Copy code

Example:

(1 − 14.72 / 64.08) × 100 = 77.0 %

yaml
Copy code

So sleep mode cuts daily energy by about 77 %.

---

### 5. Battery capacity required (Ah)

Inputs:
- Autonomy (days of no sun) → D  
- Depth of Discharge (DoD) = 0.8  
- Battery Efficiency = 0.95  
- Battery Bus Voltage (Vbus) = 12 V

Formulas:

E_total = Wh/day × D
usable_fraction = DoD × Efficiency
Battery_Ah = E_total ÷ (Vbus × usable_fraction)

makefile
Copy code

Example:

E_total = 14.72 × 2 = 29.44 Wh
usable_fraction = 0.8 × 0.95 = 0.76
Battery_Ah = 29.44 ÷ (12 × 0.76) = 4.84 Ah

yaml
Copy code

Stored energy:

4.84 Ah × 12 V = 58.1 Wh

yaml
Copy code

---

### 6. Battery pack configuration (series × parallel)

Each LiFePO₄ cell = 3.2 V, 3.2 Ah.

Series cells:

N_series = ceil(Vbus ÷ Vcell) = ceil(12.8 ÷ 3.2) = 4

yaml
Copy code

Parallel strings:

N_parallel = ceil(Required Ah ÷ Cell Ah) = ceil(4.84 ÷ 3.2) = 2

makefile
Copy code

Result:

4S2P → 8 cells total

yaml
Copy code

This means:
- 4 cells in series → 12.8 V nominal
- 2 cells in parallel → 6.4 Ah capacity  
Use a **4S LiFePO₄ BMS** rated for ≥ the system’s max current.

---

### 7. Solar panel sizing

Panel Wh/day = Load Wh/day ÷ System Efficiency
Panel W (ideal) = Panel Wh/day ÷ Sun Hours/day
Panel W (recommended) = 2 × Panel W (ideal)

java
Copy code

Example:

If daily load = 14.72 Wh, system efficiency = 65 %, sun hours = 7 h:

Panel Wh/day = 14.72 ÷ 0.65 = 22.65 Wh/day
Panel W (ideal) = 22.65 ÷ 7 = 3.24 W
Panel W (recommended) = 2 × 3.24 = 6.5 W

yaml
Copy code

So a **7–10 W panel** is safe in practice.

---

That’s how each field in the results (load, battery, and panel) is derived step-by-step.


## 🧑‍🔧 Author Notes

This tool was developed to support low-power embedded systems and smart sensor deployments (IoT nodes, weather/rain sensors, agricultural monitoring, etc.). It’s suitable for academic use, prototyping, and real field deployments.

**File name:** `index.html`  
**Tech:** Plain HTML + CSS + JavaScript (no backend)  
**License:** MIT  
**Version:** 1.0  
**Date:** 2025
