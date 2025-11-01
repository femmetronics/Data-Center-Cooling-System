# Smart Datacenter Cooling Optimization System

A Python simulation tool that optimizes datacenter cooling methods and workload scheduling to minimize environmental impact across water consumption, energy usage, and carbon emissions.

## Overview

This program simulates a smart datacenter cooling system that dynamically selects between three cooling approaches and schedules flexible workloads to reduce environmental challenges. The system considers ambient conditions, grid resource intensity, and workload constraints to make optimal decisions.

## Features

### Three Cooling Modes

1. **Cooling Towers** - Evaporative cooling with baseline efficiency
2. **Evaporative Air Cooling** - Air-side economization with evaporative assist
3. **Dry Coolers** - Zero water consumption but higher energy penalty

### Key Capabilities

- **Dynamic Mode Selection**: Automatically chooses the best cooling mode for each hour based on environmental conditions and weighted objectives
- **Flexible Workload Scheduling**: Intelligently schedules shiftable datacenter workloads (e.g., ML training) to periods with optimal conditions
- **Comprehensive Water Modeling**: Tracks onsite water consumption, blowdown, withdrawal, and offsite grid water usage
- **Carbon Accounting**: Monitors carbon emissions based on grid intensity
- **Psychrometric Calculations**: Uses Stull (2011) wet-bulb temperature approximation for realistic cooling performance

## System Model

### Environmental Inputs

Each hour includes:
- Dry-bulb temperature (°C)
- Relative humidity (%)
- Grid water consumption intensity (L/kWh)
- Grid carbon intensity (kg/kWh)

### Water System Parameters

- **Cycles of Concentration (CoC)**: Controls blowdown rates (typical: 3-7)
- **Drift Fraction**: Water lost as mist (typically ~0.2% of evaporation)
- **Blowdown Recycling**: Fraction of blowdown water recycled as makeup water

### Optimization Weights

The system minimizes a weighted objective function across:
- Onsite water consumption
- Offsite water consumption (via grid)
- Carbon emissions

## Installation

```bash
# No external dependencies required - uses Python standard library only
python smart_cooling_datacenter.py
```

**Requirements**: Python 3.7+

## Usage

### Basic Example

```python
from smart_cooling_datacenter import *

# Define environment (24 hours)
envs = demo_environment(hours=24)

# Define workload with base and flexible capacity
loads = demo_workload(hours=24)

# Configure cooling modes
cooling = build_default_cooling()

# Set water system parameters
water_sys = WaterSystemParams(
    cycles_of_concentration=5.0,
    drift_frac_of_evap=0.002,
    recycle_blowdown_frac=0.5
)

# Define optimization weights
weights = Weights(
    w_onsite_water=1.0,
    w_offsite_water=1.0,
    w_carbon=0.3
)

# Run simulation
results = simulate(
    envs, loads, cooling, water_sys, weights,
    total_flex_kWh=4000.0,
    deadline_hour=23
)

# View results
print_summary(results)
```

### Custom Configuration

```python
# Create custom cooling mode
custom_mode = CoolingMode(
    name="hybrid",
    delta_pue=0.05,
    def_wue_fn=lambda temp, rh: custom_wue_function(temp, rh),
    evaporative=True
)

# Add to cooling parameters
cooling.modes["hybrid"] = custom_mode
```

## Output

### Hourly Breakdown

For each hour, the system reports:
- Selected cooling mode
- Server energy (kWh)
- Facility energy (kWh)
- Onsite water consumption (L)
- Onsite blowdown (L)
- Onsite withdrawal (L)
- Offsite water consumption (L)
- Carbon emissions (kg)
- Flexible workload allocation (kWh)

### Summary Totals

- Total server energy
- Total facility energy
- Total onsite consumption, blowdown, and withdrawal
- Total offsite water consumption
- Total carbon emissions

### Example Output

```
=== Totals ===
Server energy (kWh):        13600
Facility energy (kWh):      15912
Onsite consumption (L):     13489
Onsite blowdown (L):        3372
Onsite withdrawal (L):      17129
Offsite water (L):          36620
Carbon (kg):                3183

=== Sample hours ===
h=10  mode=tower     T=26.6°C RH=42%  server=400kWh  onsite=631L  offsite=1083L  carbon=84.4kg  flex=0kWh
h=11  mode=tower     T=27.7°C RH=39%  server=400kWh  onsite=663L  offsite=1111L  carbon=86.5kg  flex=0kWh
```

## Algorithm Details

### Cooling Mode Selection

For each hour, the system evaluates all cooling modes using the weighted objective function:

```
Objective = w_onsite × onsite_WUE + w_offsite × offsite_WUE + w_carbon × carbon_intensity
```

The mode with the lowest objective score is selected.

### Workload Scheduling

The greedy scheduling algorithm:
1. Evaluates each hour's "cost" based on environmental conditions
2. Sorts hours by cost (ascending)
3. Allocates flexible workload to cheapest hours first
4. Respects per-hour capacity constraints
5. Ensures deadline compliance

## Configuration Parameters

### Cooling System

- **base_pue**: Baseline Power Usage Effectiveness (default: 1.15)
- **delta_pue**: Additional PUE overhead per cooling mode

### WUE Models

Each cooling mode has a Water Usage Effectiveness (WUE) function that returns liters per kWh based on temperature and humidity:

- **Cooling towers**: 0.8-3.5 L/kWh, increases with wet-bulb temperature
- **Evaporative air**: 0.1-2.5 L/kWh, optimized for hot/dry conditions
- **Dry coolers**: 0 L/kWh

## Practical Applications

- **Datacenter Planning**: Evaluate cooling strategies for different climates
- **Workload Management**: Optimize ML training schedules for sustainability
- **Resource Assessment**: Quantify water and carbon tradeoffs
- **Policy Analysis**: Model impact of water recycling and grid improvements
- **Climate Adaptation**: Test resilience under varying environmental conditions

## Limitations

- Simplified psychrometric models (Stull approximation)
- Deterministic scheduling (no uncertainty modeling)
- Illustrative WUE functions (should be calibrated to specific equipment)
- Does not model equipment degradation or maintenance
- Grid intensity profiles are synthetic

## Extending the System

### Add New Cooling Modes

```python
def adiabatic_wue_L_per_kWh(temp_c: float, rh_pct: float) -> float:
    # Your custom WUE model
    return computed_wue

new_mode = CoolingMode(
    name="adiabatic",
    delta_pue=0.03,
    def_wue_fn=adiabatic_wue_L_per_kWh,
    evaporative=True
)
```

### Custom Objective Weights

Adjust weights to prioritize different metrics:
- Water-scarce regions: increase `w_onsite_water`
- Carbon-intensive grids: increase `w_carbon`
- Holistic optimization: balance all weights equally

## References

- Stull, R. (2011). Wet-Bulb Temperature from Relative Humidity and Air Temperature. *Journal of Applied Meteorology and Climatology*, 50(11), 2267-2269.
- ASHRAE TC 9.9 - Mission Critical Facilities, Technology Spaces, and Electronic Equipment
- EPA Guidelines for Water Efficiency in Data Centers

## License

This code is provided as-is for educational and research purposes.

## Contributing

Contributions welcome! Areas for enhancement:
- Real-world equipment WUE calibration
- Stochastic workload models
- Multi-day optimization with storage
- Geographic-specific grid profiles
- Machine learning for predictive scheduling

## Contact

For questions or collaboration opportunities, please open an issue or submit a pull request.

## Code created by Ashley and Manha
