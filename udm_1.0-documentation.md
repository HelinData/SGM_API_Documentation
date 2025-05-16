# Helin - Universal Data Model (UDM) 1.0
## Comprehensive Documentation

---

## Table of Contents

1. [Introduction](#introduction)
2. [Asset Naming Convention](#asset-naming-convention)
3. [Device Categories](#device-categories)
4. [Data Model Overview](#data-model-overview)
5. [Model Specifications](#model-specifications)
   - [Inverter Models](#inverter-models)
   - [Inverter Control Models](#inverter-control-models)
   - [Metering Models](#metering-models)
   - [Environmental Models](#environmental-models)
   - [Storage Models](#storage-models)
6. [Common Parameters](#common-parameters)
7. [Parameter Categories](#parameter-categories)
8. [Implementation Guidelines](#implementation-guidelines)
9. [Appendix: Full Model Reference](#appendix-full-model-reference)

---

## Introduction

The Universal Data Model (UDM) provides a standardized framework for representing and communicating data in energy management systems, particularly for renewable energy installations. This data model is based on SunSpec Alliance standards and provides a comprehensive set of models for various devices and measurements commonly found in solar, storage, and other renewable energy systems.

The UDM 1.0 enables consistent data representation across different manufacturers, simplifies integration with energy management systems, and provides a foundation for analytics, monitoring, and control applications.

---

## Asset Naming Convention

The UDM 1.0 utilizes a structured naming convention for assets to ensure consistent identification across systems:

**Format:** `Client/SiteID/PlantID/system/DeviceCat_ID/measurement`

**Example:** `SRCK/DRDN090/PL01/SYS01/INVR_AB8383/TmpCab`

This hierarchical structure allows for easy identification of:
- Client or organization
- Site and plant identification
- System identifier
- Device category and unique ID
- Specific measurement point

---

## Device Categories

The UDM 1.0 defines several device categories that are used in asset naming:

| Category | Meaning | Description |
|----------|---------|-------------|
| INVR | Inverter | Solar inverter or power conversion system |
| LOGG | Logger | Data acquisition and logging system |
| PMTR | Power Meter | Electrical measurement device |
| EMS | Energy Management System | Control system for energy optimization |
| EVSE | Electric Vehicle Charger Socket | EV charging infrastructure |
| BESS | Battery Energy Storage System | Energy storage solution |
| MEAS | Measurement | Generic measurement device |
| RIO | Remote Input/Output | Remote IO modules |

---

## Data Model Overview

The UDM 1.0 consists of 12 primary models that define different aspects of renewable energy systems:

1. **Inverter Models** - Define electrical characteristics and measurements for inverters
   - Model 101: Single-phase Inverter
   - Model 102: Split-phase Inverter
   - Model 103: Three-phase Inverter

2. **Inverter Control Models** - Define settings and controls for inverters
   - Model 120: Inverter Controls Nameplate Ratings
   - Model 121: Inverter Controls Basic Settings
   - Model 122: Inverter Controls Extended Measurements and Status
   - Model 123: Immediate Inverter Controls
   - Model 124: Basic Storage Controls

3. **Metering Models** - Define electrical measurements from power meters
   - Model 203: Three-Phase Wye (abcn) meter

4. **Environmental Models** - Define environmental measurements
   - Model 302: Irradiance
   - Model 307: Base Meteorological Model

5. **Storage Models** - Define characteristics and controls for storage systems
   - Model 802: Battery Base Model

Each model is defined with a unique identifier (ID) and a set of parameters with specific data types, units, and scale factors to ensure accurate representation of values.

---

## Model Specifications

### Inverter Models

#### Model 101: Single-phase Inverter
A model representing measurements from a single-phase inverter.

**Key Parameters:**
- AC Current and voltage measurements
- Power measurements (active, reactive, apparent)
- Frequency
- Temperature readings
- Status indicators

**Example Parameters:**
```
A: AC Current (A)
AphA: Phase A Current (A)
PPVphAB: Phase Voltage AB (V)
```

**Total Parameters:** 45

#### Model 102: Split-phase Inverter
A model representing measurements from a split-phase inverter.

**Key Parameters:**
- AC Current measurements
- Voltage measurements
- Power measurements
- Frequency
- Scale factors

**Example Parameters:**
```
A: AC Total Current (Amps)
PhV: AC Voltage (L-N) (Volts)
```

**Total Parameters:** 22

#### Model 103: Three-phase Inverter
A model representing measurements from a three-phase inverter.

**Key Parameters:**
- AC Current measurements for all phases
- Voltage measurements
- Power measurements (per phase and total)
- Frequency
- Temperature readings
- Status indicators

**Example Parameters:**
```
A: AC Current (Amps)
AphA: Phase A Current (Amps)
AphB: Phase B Current (Amps)
```

**Total Parameters:** 42

### Inverter Control Models

#### Model 120: Inverter Controls Nameplate Ratings
Defines the nameplate capabilities of the inverter.

**Key Parameters:**
- DER type identification
- Power ratings
- Voltage ratings
- Current ratings

**Example Parameters:**
```
DERTyp: Type of DER device
WRtg: Continuous power output capability (W)
VARtg: Continuous Volt-Ampere capability (VA)
```

**Total Parameters:** 18

#### Model 121: Inverter Controls Basic Settings
Defines the basic control settings for an inverter.

**Key Parameters:**
- Maximum power output settings
- Voltage reference settings
- Power factor settings
- Scale factors

**Example Parameters:**
```
WMax: Maximum Power Output (W)
VRef: Point of Common Coupling Voltage (V)
VRefOfs: Voltage Offset (V)
```

**Total Parameters:** 32

#### Model 122: Inverter Controls Extended Measurements and Status
Provides extended status and measurement information for inverter controls.

**Key Parameters:**
- Connection status indicators
- Voltage measurements
- Current measurements
- Scale factors

**Example Parameters:**
```
PVConn: PV inverter present/available status
StorConn: Storage inverter present/available status
ECPConn: ECP connection status
```

**Total Parameters:** 22

#### Model 123: Immediate Inverter Controls
Provides real-time control capabilities for inverters.

**Key Parameters:**
- Connection controls
- Power control settings
- Voltage control settings
- Scale factors

**Example Parameters:**
```
Conn_WinTms: Time window for connect/disconnect (Seconds)
Conn_RvrtTms: Timeout period for connect/disconnect (Seconds)
Conn: Connection control
```

**Total Parameters:** 26

#### Model 124: Basic Storage Controls
Provides control capabilities specific to storage inverters.

**Key Parameters:**
- Charge status indicators
- Battery voltage measurements
- Power control settings
- Current control settings
- Control parameters

**Example Parameters:**
```
ChaSt: Charge Status
ChaStVnd: Vendor Charge Status
InBatV: Internal Battery Voltage (V)
```

**Total Parameters:** 22

### Metering Models

#### Model 203: Three-Phase Wye (abcn) meter
Represents measurements from a three-phase wye-connected power meter.

**Key Parameters:**
- Current measurements (per phase and total)
- Voltage measurements (multiple configurations)
- Power measurements (active, reactive, apparent)
- Frequency measurement
- Scale factors

**Example Parameters:**
```
A: Total AC Current (A)
AphA: Phase A Current (A)
AphB: Phase B Current (A)
```

**Total Parameters:** 57

### Environmental Models

#### Model 302: Irradiance
Provides measurements for solar irradiance.

**Key Parameters:**
- Global horizontal irradiance
- Plane-of-array irradiance
- Diffuse irradiance

**Example Parameters:**
```
GHI: Global Horizontal Irradiance (W/m2)
POAI: Plane-of-Array Irradiance (W/m2)
DFI: Diffuse Irradiance (W/m2)
```

**Total Parameters:** 7

#### Model 307: Base Meteorological Model
Provides basic meteorological measurements.

**Key Parameters:**
- Ambient temperature
- Relative humidity
- Barometric pressure
- Wind measurements

**Example Parameters:**
```
TmpAmb: Ambient Temperature (C)
RH: Relative Humidity (Pct)
Pres: Barometric Pressure (HPa)
```

**Total Parameters:** 13

### Storage Models

#### Model 802: Battery Base Model
Provides measurements and controls for battery energy storage systems.

**Key Parameters:**
- Battery capacity ratings
- Charge/discharge power ratings
- Voltage measurements
- Current measurements
- State of charge indicators
- Control parameters
- Scale factors

**Example Parameters:**
```
AHRtg: Nameplate charge capacity (Ah)
WHRtg: Nameplate energy capacity (Wh)
WChaRteMax: Maximum rate of energy transfer into storage (W)
```

**Total Parameters:** 58

---

## Common Parameters

Certain parameters appear across multiple models, creating a consistent framework for data representation. These common parameters include:

1. **Model Identifiers (All Models)**
   - ID: Model identifier
   - L: Model length (in registers)

2. **Current Measurements (Models 101, 102, 103, 203, 802)**
   - A: Total AC Current
   - A_SF: Scale factor for current

3. **Power Measurements (Models 101, 102, 103, 203, 802)**
   - W: Active power
   - W_SF: Scale factor for active power

4. **Frequency Measurements (Models 101, 102, 103, 203)**
   - Hz: Frequency
   - Hz_SF: Scale factor for frequency

5. **Voltage Measurements (Multiple Models)**
   - Various voltage parameters with model-specific implementations

These common parameters make it easier to integrate data from different models and devices, ensuring consistency in representation and interpretation.

---

## Parameter Categories

The parameters in the UDM 1.0 can be categorized by their function:

### Voltage Parameters
Measurements related to electrical voltage, typically in Volts (V).

### Current Parameters
Measurements related to electrical current, typically in Amperes (A).

### Power Parameters
Measurements related to electrical power, including:
- Active Power (W)
- Reactive Power (VAr)
- Apparent Power (VA)

### Frequency Parameters
Measurements related to electrical frequency, typically in Hertz (Hz).

### Temperature Parameters
Measurements related to temperature, typically in Celsius (°C).

### Status Parameters
Indicators of operational status, typically represented as enumerated values.

### Control Parameters
Settings that influence the behavior of devices, typically represented as setpoints.

### Scale Factors
Parameters that provide scaling information for other measurements, allowing for efficient data representation.

### Model Identifiers
Parameters that identify the model and its structure.

---

## Implementation Guidelines

When implementing the UDM, consider the following guidelines:

1. **Asset Naming**
   - Follow the prescribed naming convention to ensure consistent identification across systems
   - Maintain uniqueness of asset identifiers within the system

2. **Parameter Scaling**
   - Scale factors (parameters with names ending in "_SF") are critical for interpreting measurement values correctly
   - Scale factors are typically signed integers
   - The actual value is calculated as: `raw_value × 10^(scale_factor)`
   - Each scale factor applies to all parameters with the matching prefix
   - Example: `A_SF` applies to `A`, `AphA`, `AphB`, `AphC`, etc.
   - Example: If `A` has a value of 1534 and `A_SF` has a value of -1, then the actual current is 1534 × 10^(-1) = 153.4 A

3. **Data Types**
   - Integer values: typically stored as 16-bit or 32-bit values
   - Scale factors: typically stored as 16-bit signed integers
   - Status values: typically enumerated as 16-bit unsigned integers

4. **Scale Factor Applications**
   - Current parameters (A*): Use `A_SF` for scaling
   - Voltage parameters (V*, PhV*, PPV*): Use `V_SF` for scaling
   - Power parameters (W*): Use `W_SF` for scaling
   - Frequency parameters (Hz*): Use `Hz_SF` for scaling
   - Apparent power parameters (VA*): Use `VA_SF` for scaling
   - Reactive power parameters (VAr*): Use `VAr_SF` for scaling
   - Power factor parameters (PF*): Use `PF_SF` for scaling
   - Energy parameters (WH*): Use `WH_SF` for scaling
   - Temperature parameters (Tmp*): Use `Tmp_SF` for scaling
   - DC current parameters (DCA*): Use `DCA_SF` for scaling
   - DC voltage parameters (DCV*): Use `DCV_SF` for scaling
   - DC power parameters (DCW*): Use `DCW_SF` for scaling

5. **Integration**
   - Ensure consistent interpretation of units and scale factors
   - Validate data against expected ranges
   - Implement appropriate error handling for missing or invalid data
   - Always apply scale factors before displaying or processing measurement values

---

## Appendix: Full Model Reference

### Complete Parameter Listings

#### Model 101: Single-phase Inverter

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| A | A | AC Current |
| AphA | A | Phase A Current |
| AphB | A | Phase B Current |
| AphC | A | Phase C Current |
| A_SF | - | Scale factor for current |
| PPVphAB | V | Phase Voltage AB |
| PPVphBC | V | Phase Voltage BC |
| PPVphCA | V | Phase Voltage CA |
| PhVphA | V | Phase Voltage AN |
| PhVphB | V | Phase Voltage BN |
| PhVphC | V | Phase Voltage CN |
| V_SF | - | Scale factor for voltage |
| W | W | AC Power |
| W_SF | - | Scale factor for power |
| Hz | Hz | Line Frequency |
| Hz_SF | - | Scale factor for frequency |
| VA | VA | AC Apparent Power |
| VA_SF | - | Scale factor for apparent power |
| VAr | var | AC Reactive Power |
| VAr_SF | - | Scale factor for reactive power |
| PF | Pct | AC Power Factor |
| PF_SF | - | Scale factor for power factor |
| WH | Wh | AC Energy |
| WH_SF | - | Scale factor for energy |
| DCA | A | DC Current |
| DCA_SF | - | Scale factor for DC current |
| DCV | V | DC Voltage |
| DCV_SF | - | Scale factor for DC voltage |
| DCW | W | DC Power |
| DCW_SF | - | Scale factor for DC power |
| TmpCab | C | Cabinet Temperature |
| TmpSnk | C | Heat Sink Temperature |
| TmpTrns | C | Transformer Temperature |
| TmpOt | C | Other Temperature |
| Tmp_SF | - | Scale factor for temperature |
| St | - | Enumerated value. Operating state |
| StVnd | - | Vendor specific operating state code |
| Evt1 | - | Bitmask value. Event fields |
| Evt2 | - | Reserved for future use |
| EvtVnd1 | - | Vendor defined events |
| EvtVnd2 | - | Vendor defined events |
| EvtVnd3 | - | Vendor defined events |
| EvtVnd4 | - | Vendor defined events |

#### Model 102: Split-phase Inverter

| Parameter | Unit | Description |
|-----------|------|-------------|
| A | Amps | AC Total Current |
| A_SF | - | AC Current Scale Factor |
| PhV | Volts | AC Voltage (L-N) |
| PhV_SF | - | AC Voltage Scale Factor |
| PhVAB | Volts | AC Voltage (L1-L2) |
| W | Watts | AC Power |
| W_SF | - | AC Power Scale Factor |
| Hz | Hertz | AC Frequency |
| Hz_SF | - | AC Frequency Scale Factor |
| VA | Volt-Amps | Apparent Power |
| VA_SF | - | Apparent Power Scale Factor |
| VAr | VAr | Reactive Power |
| VAr_SF | - | Reactive Power Scale Factor |
| PF | % | Power Factor |
| PF_SF | - | Power Factor Scale Factor |
| WH | Watt-Hours | Energy Output |
| WH_SF | - | Energy Output Scale Factor |
| DCA | Amps | DC Current |
| DCA_SF | - | DC Current Scale Factor |
| DCV | Volts | DC Voltage |
| DCV_SF | - | DC Voltage Scale Factor |
| DCW | Watts | DC Power |

#### Model 103: Three-phase Inverter

| Parameter | Unit | Description |
|-----------|------|-------------|
| A | Amps | AC Current |
| AphA | Amps | Phase A Current |
| AphB | Amps | Phase B Current |
| AphC | Amps | Phase C Current |
| A_SF | - | AC Current Scale Factor |
| PhVphA | Volts | Phase A Voltage |
| PhVphB | Volts | Phase B Voltage |
| PhVphC | Volts | Phase C Voltage |
| PhV_SF | - | AC Voltage Scale Factor |
| W | Watts | AC Power |
| W_SF | - | AC Power Scale Factor |
| Hz | Hertz | AC Frequency |
| Hz_SF | - | AC Frequency Scale Factor |
| VA | Volt-Amps | Apparent Power |
| VA_SF | - | Apparent Power Scale Factor |
| VAr | VAr | Reactive Power |
| VAr_SF | - | Reactive Power Scale Factor |
| PF | % | Power Factor |
| PF_SF | - | Power Factor Scale Factor |
| WH | Watt-Hours | Energy Output Accumnaltive |
| WHD | Watt-Hours | Energy Output Daily |
| WHM | Watt-Hours | Energy Output Monthly |
| WH_SF | - | Energy Output Scale Factor |
| TRT |  | Total Running Time |
| DCA | Amps | DC Current |
| DCA_SF | - | DC Current Scale Factor |
| DCV | Volts | DC Voltage |
| DCV_SF | - | DC Voltage Scale Factor |
| DCW | Watts | DC Power |
| DCW_SF | - | DC Power Scale Factor |
| TmpCab | °C | Cabinet Temperature |
| TmpSnk | °C | Heat Sink Temperature |
| TmpTrns | °C | Transformer Temperature |
| TmpOt | °C | Other Temperature |
| St | - | Operating State |
| StVnd | - | Vendor Operating State |
| Evt1 | - | Event Flags 1 |
| Evt2 | - | Event Flags 2 |
| EvtVnd1 | - | Vendor Event Flags 1 |
| EvtVnd2 | - | Vendor Event Flags 2 |
| EvtVnd3 | - | Vendor Event Flags 3 |
| EvtVnd4 | - | Vendor Event Flags 4 |

#### Model 120: Inverter Controls Nameplate Ratings

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| DERTyp | - | Type of DER device. Default value is 4 to indicate PV device. |
| WRtg | W | Continuous power output capability of the inverter. |
| VARtg | VA | Continuous Volt-Ampere capability of the inverter. |
| VArRtgQ1 | VAr | Continuous VAR capability of the inverter in quadrant 1. |
| VArRtgQ2 | VAr | Continuous VAR capability of the inverter in quadrant 2. |
| VArRtgQ3 | VAr | Continuous VAR capability of the inverter in quadrant 3. |
| VArRtgQ4 | VAr | Continuous VAR capability of the inverter in quadrant 4. |
| ARtg | A | Maximum RMS AC current level capability of the inverter. |
| PFRtgQ1 | cos() | Minimum power factor capability of the inverter in quadrant 1. |
| PFRtgQ2 | cos() | Minimum power factor capability of the inverter in quadrant 2. |
| PFRtgQ3 | cos() | Minimum power factor capability of the inverter in quadrant 3. |
| PFRtgQ4 | cos() | Minimum power factor capability of the inverter in quadrant 4. |
| WHRtg | Wh | Nominal energy rating of storage device. |
| AhrRtg | AH | The usable capacity of the battery. |
| MaxChaRte | W | Maximum rate of energy transfer into the storage device. |
| MaxDisChaRte | W | Maximum rate of energy transfer out of the storage device. |

#### Model 121: Inverter Controls Basic Settings

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model Identifier |
| L | registers | Model Length |
| WMax | W | Maximum Power Output |
| VRef | V | Point of Common Coupling Voltage |
| VRefOfs | V | Voltage Offset |
| VMax | V | Maximum Voltage Setpoint |
| VMin | V | Minimum Voltage Setpoint |
| VAMax | VA | Maximum Apparent Power Setpoint |
| VArMaxQ1 | var | Maximum Reactive Power Q1 |
| VArMaxQ2 | var | Maximum Reactive Power Q2 |
| VArMaxQ3 | var | Maximum Reactive Power Q3 |
| VArMaxQ4 | var | Maximum Reactive Power Q4 |
| WGra | % WMax/sec | Active Power Ramp Rate |
| PFMinQ1 | cos() | Minimum Power Factor Q1 |
| PFMinQ2 | cos() | Minimum Power Factor Q2 |
| PFMinQ3 | cos() | Minimum Power Factor Q3 |
| PFMinQ4 | cos() | Minimum Power Factor Q4 |
| VArAct | - | VAR Action on Charge/Discharge |
| ClcTotVA | - | Total Apparent Power Calculation Method |
| MaxRmpRte | % WGra | Maximum Ramp Rate Setpoint |
| ECPNomHz | Hz | Nominal Frequency Setpoint |
| ConnPh | - | Connected Phase Identity |
| WMax_SF | - | Scale Factor for Real Power |
| VRef_SF | - | Scale Factor for PCC Voltage |
| VRefOfs_SF | - | Scale Factor for Offset Voltage |
| VMinMax_SF | - | Scale Factor for Min/Max Voltage |
| VAMax_SF | - | Scale Factor for Apparent Power |
| VArMax_SF | - | Scale Factor for Reactive Power |
| WGra_SF | - | Scale Factor for Ramp Rate |
| PFMin_SF | - | Scale Factor for Min Power Factor |
| MaxRmpRte_SF | - | Scale Factor for Max Ramp Rate |
| ECPNomHz_SF | - | Scale Factor for Nominal Frequency |

#### Model 122: Inverter Controls Extended Measurements and Status

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| PVConn | - | PV inverter present/available status. |
| StorConn | - | Storage inverter present/available status. |
| ECPConn | - | ECP connection status: disconnected=0 connected=1. |
| ActWh | Wh | AC lifetime active (real) energy output. |
| ActVAh | VAh | AC lifetime apparent energy output. |
| ActVArhQ1 | varh | AC lifetime reactive energy output in quadrant 1. |
| ActVArhQ2 | varh | AC lifetime reactive energy output in quadrant 2. |
| ActVArhQ3 | varh | AC lifetime negative energy output in quadrant 3. |
| ActVArhQ4 | varh | AC lifetime reactive energy output in quadrant 4. |
| VArAval | var | Amount of VARs available without impacting watts output. |
| VArAval_SF | - | Scale factor for available VARs. |
| WAval | var | Amount of Watts available. |
| WAval_SF | - | Scale factor for available Watts. |
| StSetLimMsk | - | Bit Mask indicating setpoint limit(s) reached. |
| StActCtl | - | Bit Mask indicating which inverter controls are currently active. |
| TmSrc | - | Source of time synchronization. |
| Tms | Secs | Seconds since 01-01-2000 00:00 UTC. |
| RtSt | - | Bit Mask indicating active ride-through status. |
| Ris | ohms | Isolation resistance. |
| Ris_SF | - | Scale factor for isolation resistance. |

#### Model 123: Immediate Inverter Controls

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| Conn_WinTms | Seconds | Time window for connect/disconnect. |
| Conn_RvrtTms | Seconds | Timeout period for connect/disconnect. |
| Conn | - | Enumerated valued. Connection control. |
| WMaxLimPct | % WMax | Set power output to specified level. |
| WMaxLimPct_WinTms | Seconds | Time window for power limit change. |
| WMaxLimPct_RvrtTms | Seconds | Timeout period for power limit. |
| WMaxLimPct_RmpTms | Seconds | Ramp time for moving from current setpoint to new setpoint. |
| WMaxLimEna | - | Enumerated valued. Throttle enable/disable control. |
| OutPFSet_Ena | - | Enumerated valued. Fixed power factor enable/disable control. |
| OutPFSet | cos() | Set power factor to specific value - cosine of angle. |
| OutPFSet_WinTms | Seconds | Time window for power factor change. |
| OutPFSet_RvrtTms | Seconds | Timeout period for power factor. |
| OutPFSet_RmpTms | Seconds | Ramp time for moving from current setpoint to new setpoint. |
| VArWMaxPct | % WMax | Reactive power in percent of WMax. |
| VArMaxPct | % VArMax | Reactive power in percent of VArMax. |
| VArAvalPct | % VArAval | Reactive power in percent of VArAval. |
| VArPct_WinTms | Seconds | Time window for VAR limit change. |
| VArPct_RvrtTms | Seconds | Timeout period for VAR limit. |
| VArPct_RmpTms | Seconds | Ramp time for moving from current setpoint to new setpoint. |
| VArPct_Mod | - | Enumerated value. VAR percent limit mode. |
| VArPct_Ena | - | Enumerated valued. Percent limit VAr enable/disable control. |
| WMaxLimPct_SF | - | Scale factor for power output percent. |
| OutPFSet_SF | - | Scale factor for power factor. |
| VArPct_SF | - | Scale factor for reactive power percent. |

#### Model 124: Basic Storage Controls

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model Identifier |
| L | registers | Model Length |
| ChaSt | - | Charge Status |
| ChaStVnd | - | Vendor Charge Status |
| InBatV | V | Internal Battery Voltage |
| ChaMaxW | W | Maximum Charge Power |
| ChaMaxVAr | VAr | Maximum Charge Reactive Power |
| ChaMaxA | A | Maximum Charge Current |
| DisChaMaxW | W | Maximum Discharge Power |
| DisChaMaxVAr | VAr | Maximum Discharge Reactive Power |
| DisChaMaxA | A | Maximum Discharge Current |
| StorCtl_Mod | - | Storage Control Mode |
| ChaCtl | - | Charge Control |
| DisChaCtl | - | Discharge Control |
| StorCtl_Mod_SF | - | Scale Factor for Storage Control Mode |
| InBatV_SF | - | Scale Factor for Internal Battery Voltage |
| ChaMaxW_SF | - | Scale Factor for Maximum Charge Power |
| ChaMaxVAr_SF | - | Scale Factor for Maximum Charge Reactive Power |
| ChaMaxA_SF | - | Scale Factor for Maximum Charge Current |
| DisChaMaxW_SF | - | Scale Factor for Maximum Discharge Power |
| DisChaMaxVAr_SF | - | Scale Factor for Maximum Discharge Reactive Power |
| DisChaMaxA_SF | - | Scale Factor for Maximum Discharge Current |

#### Model 203: Three-Phase Wye (abcn) meter

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| A | A | Total AC Current |
| AphA | A | Phase A Current |
| AphB | A | Phase B Current |
| AphC | A | Phase C Current |
| A_SF | - | Current scale factor |
| PhV | V | Line to Neutral AC Voltage |
| PhVphA | V | Phase Voltage AN |
| PhVphB | V | Phase Voltage BN |
| PhVphC | V | Phase Voltage CN |
| PPV | V | Line to Line AC Voltage |
| PhVphAB | V | Phase Voltage AB |
| PhVphBC | V | Phase Voltage BC |
| PhVphCA | V | Phase Voltage CA |
| V_SF | - | Voltage scale factor |
| Hz | Hz | Frequency |
| Hz_SF | - | Frequency scale factor |
| W | W | Total Real Power |
| WphA | W | Watts phase A |
| WphB | W | Watts phase B |
| WphC | W | Watts phase C |
| W_SF | - | Real Power scale factor |
| VA | VA | AC Apparent Power |
| VAphA | VA | VA phase A |
| VAphB | VA | VA phase B |
| VAphC | VA | VA phase C |
| VA_SF | - | Apparent Power scale factor |
| VAR | var | Reactive Power |
| VARphA | var | VAR phase A |
| VARphB | var | VAR phase B |
| VARphC | var | VAR phase C |
| VAR_SF | - | Reactive Power scale factor |
| PF | Pct | Power Factor |
| PFphA | Pct | PF phase A |
| PFphB | Pct | PF phase B |
| PFphC | Pct | PF phase C |
| PF_SF | - | Power Factor scale factor |
| TotWhExp | Wh | Total Real Energy Exported |
| TotWhExpPhA | Wh | Total Watt-hours Exported phase A |
| TotWhExpPhB | Wh | Total Watt-hours Exported phase B |
| TotWhExpPhC | Wh | Total Watt-hours Exported phase C |
| TotWhImp | Wh | Total Real Energy Imported |
| TotWhImpPhA | Wh | Total Watt-hours Imported phase A |
| TotWhImpPhB | Wh | Total Watt-hours Imported phase B |
| TotWhImpPhC | Wh | Total Watt-hours Imported phase C |
| TotWh_SF | - | Real Energy scale factor |
| TotVAhExp | VAh | Total Apparent Energy Exported |
| TotVAhExpPhA | VAh | Total VA-hours Exported phase A |
| TotVAhExpPhB | VAh | Total VA-hours Exported phase B |
| TotVAhExpPhC | VAh | Total VA-hours Exported phase C |
| TotVAhImp | VAh | Total Apparent Energy Imported |
| TotVAhImpPhA | VAh | Total VA-hours Imported phase A |
| TotVAhImpPhB | VAh | Total VA-hours Imported phase B |
| TotVAhImpPhC | VAh | Total VA-hours Imported phase C |
| TotVAh_SF | - | Apparent Energy scale factor |
| Evt | - | Meter Event Flags |

#### Model 302: Irradiance

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| GHI | W/m2 | Global Horizontal Irradiance |
| POAI | W/m2 | Plane-of-Array Irradiance |
| DFI | W/m2 | Diffuse Irradiance |
| DNI | W/m2 | Direct Normal Irradiance |
| OTI | W/m2 | Other Irradiance |

#### Model 307: Base Meteorological Model

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | registers | Model length |
| TmpAmb | C | Ambient Temperature |
| RH | Pct | Relative Humidity |
| Pres | HPa | Barometric Pressure |
| WndSpd | mps | Wind Speed |
| WndDir | deg | Wind Direction |
| Rain | mm | Rainfall |
| Snw | mm | Snow Depth |
| PPT | - | Precipitation Type (WMO 4680 SYNOP code reference) |
| ElecFld | Vm | Electric Field |
| SurWet | kO | Surface Wetness |
| SoilWet | Pct | Soil Wetness |

#### Model 802: Battery Base Model

| Parameter | Unit | Description |
|-----------|------|-------------|
| ID | - | Model identifier |
| L | - | Model length |
| AHRtg | registers | Nameplate charge capacity in amp-hours. |
| WHRtg | Ah | Nameplate energy capacity in DC watt-hours. |
| WChaRteMax | Wh | Maximum rate of energy transfer into the storage device in DC watts. |
| WDisChaRteMax | W | Maximum rate of energy transfer out of the storage device in DC watts. |
| DisChaRte | W | Self discharge rate. Percentage of capacity discharged per day. |
| SoCMax | %WHRtg | Manufacturer maximum state of charge, expressed as a percentage. |
| SoCMin | %WHRtg | Manufacturer minimum state of charge, expressed as a percentage. |
| SocRsvMax | %WHRtg | Setpoint for maximum reserve for storage as a percentage. |
| SoCRsvMin | %WHRtg | Setpoint for minimum reserve for storage as a percentage. |
| SoC | %WHRtg | State of charge, expressed as a percentage. |
| DoD | %WHRtg | Depth of discharge, expressed as a percentage. |
| SoH | % | Percentage of battery life remaining. |
| NCyc | % | Number of cycles executed in the battery. |
| ChaSt | - | Charge status of storage device. |
| LocRemCtl | - | Battery control mode. |
| Hb | - | Value incremented every second with periodic resets to zero. |
| CtrlHb | - | Value incremented every second with periodic resets to zero. |
| AlmRst | - | Used to reset any latched alarms. |
| Typ | - | Type of battery. |
| State | - | State of the battery bank. |
| StateVnd | - | Vendor specific battery bank state. |
| WarrDt | - | Date the device warranty expires. |
| Evt1 | - | Alarms and warnings. Bit flags. |
| Evt2 | - | Alarms and warnings. Bit flags. |
| EvtVnd1 | - | Vendor defined events. |
| EvtVnd2 | - | Vendor defined events. |
| V | - | DC Bus Voltage. |
| VMax | V | Instantaneous maximum battery voltage. |
| VMin | V | Instantaneous minimum battery voltage. |
| CellVMax | V | Maximum voltage for all cells in the bank. |
| CellVMaxStr | V | String containing the cell with maximum voltage. |
| CellVMaxMod | - | Module containing the cell with maximum voltage. |
| CellVMin | - | Minimum voltage for all cells in the bank. |
| CellVMinStr | V | String containing the cell with minimum voltage. |
| CellVMinMod | - | Module containing the cell with minimum voltage. |
| CellVAvg | - | Average cell voltage for all cells in the bank. |
| A | V | Total DC current flowing to/from the battery bank. |
| AChaMax | A | Instantaneous maximum DC charge current. |
| ADisChaMax | A | Instantaneous maximum DC discharge current. |
| W | A | Total power flowing to/from the battery bank. |
| ReqInvState | W | Request from battery to start or stop the inverter. |
| ReqW | - | AC Power requested by battery. |
| SetOp | W | Instruct the battery bank to perform an operation. |
| SetInvState | - | Set the current state of the inverter. |
| AHRtg_SF | - | Scale factor for charge capacity. |
| WHRtg_SF | - | Scale factor for energy capacity. |
| WChaDisChaMax_SF | - | Scale factor for max charge/discharge rate. |
| DisChaRte_SF | - | Scale factor for self discharge rate. |
| SoC_SF | - | Scale factor for state of charge values. |
| DoD_SF | - | Scale factor for depth of discharge. |
| SoH_SF | - | Scale factor for state of health. |
| V_SF | - | Scale factor for DC bus voltage. |
| CellV_SF | - | Scale factor for cell voltage. |
| A_SF | - | Scale factor for DC current. |
| AMax_SF | - | Scale factor for instantaneous charge/discharge current. |
| W_SF | - | Scale factor for AC power request. |

---

*This documentation was created based on UDM_1.0.xlsx data model specifications. It provides a comprehensive reference guide to the Universal Data Model 1.0, including all parameters and their specifications.*
