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
