# gbXML SWH Extension — DOE-2.2 / eQUEST Mapping Guide

**Companion to:** gbXML Service Water Heating Extension Schema v0.3 (merged)
**Engine:** DOE-2.2 (via eQUEST or direct BDL input)
**Date:** June 2026

---

## 1. Overview

DOE-2.2 models domestic hot water at the loop level through CIRCULATION-LOOP (TYPE=DHW) with one or more DW-HEATER objects. Demand is an hourly flow profile on the loop. Equipment is characterized by Heat Input Ratio (HIR = 1/efficiency), tank volume, and tank UA. Distribution losses are modeled as an aggregate loop-level quantity, not per-segment geometry. DOE-2.2 is a **loop-aggregate, hourly-schedule, simplified-equipment** engine.

## 2. Element Mapping

### 2.1 Demand Side

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| ServiceWaterLoad | DHW load on CIRCULATION-LOOP | Exporter sums all Space loads into one loop demand. |
| WaterUseValue (VolumeFlowRate) | Peak flow (GPM) | Multiplied by number of spaces served. |
| WaterUseScheduleIdRef | SCHEDULE (TYPE=FRACTION) | Hourly fractions modulating peak flow. |
| HotWaterUseFraction | (fraction of flow that is hot) | Applied to peak flow. |
| ColdWaterLoad | **Not consumed** | DOE-2.2 does not model cold water energy. |
| DrawProfileIdRef | **Not consumed** | DOE-2.2 uses hourly schedules. |
| ZoneAggregateDHWDemand | **Not consumed** | DOE-2.2 sums per-Space loads or takes a single loop demand. |
| ServiceWaterZone | No direct equivalent | Organizational only. |

### 2.2 Supply Side (Equipment)

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| WaterHeater | DW-HEATER | One per loop. Multiple schema WaterHeaters may need to be combined. |
| WaterHeaterType | DW-HEATER TYPE | Storage→STORAGE-HEATER, Instantaneous→INSTANTANEOUS-HEATER, HeatPump→HEAT-PUMP, HotWaterSupplyBoiler→BOILER. |
| FuelSource | FUEL-METER | Determines which meter is charged. |
| InputCapacity | CAPACITY (Btu/h) | Direct mapping. |
| ThermalEfficiency | HEAT-INPUT-RATIO | HIR = 1 / ThermalEfficiency. |
| UniformEnergyFactor | ENERGY-FACTOR or HIR | UEF approximated to EF; HIR = 1/UEF for simplified path. |
| EnergyFactor | ENERGY-FACTOR | Direct for legacy equipment. |
| SystemSeasonalEfficiency | **Primary for DOE-2.2** | HIR = 1 / SystemSeasonalEfficiency. Preferred annualized value. |
| SetpointTemperature | AQUASTAT-SETPT | Tank/heater setpoint (°F). |
| CompressorZoneIdRef | **Limited** | DOE-2.2 has limited HPWH zone-source modeling. |
| SourceAirConfiguration | **Informational** | DOE-2.2 does not model ducted HPWH sources. |
| HPWHProductModelDesignation | **Informational** | DOE-2.2 uses simplified COP. |
| SourceHydronicLoopIdRef | **Informational** | DOE-2.2 indirect heater modeling is limited. |
| BackupHeaterFuelSource | (backup fuel on DW-HEATER) | Backup element fuel. |
| BackupInputCapacity | (backup capacity) | Backup element sizing. |

### 2.3 Storage Tanks

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| StorageTank (Primary) | Part of DW-HEATER | TANK-VOLUME + TANK-UA on the heater. |
| StorageVolume | TANK-VOLUME | Gallons. |
| StandbyLossCoefficient | TANK-UA | Btu/h-°F. |
| TankZoneIdRef | **Limited** | DOE-2.2 uses a simplified ambient assumption. |
| StorageTank (Swing) | Additional TANK-VOLUME | DOE-2.2 cannot model staging; exporter combines volumes. |
| StorageTank (Preheat) | **Limited** | Solar preheat modeled via SOLAR-FRACTION keyword. |
| TankRole | **Informational** | DOE-2.2 models one aggregate tank per heater. |
| LinkedPrimaryTankIdRef | **Informational** | Staging not representable. |

### 2.4 HPWH Performance

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| HPWHCapacityValidationPoints | **Not consumed** | DOE-2.2 uses simplified COP model. |
| HPWHPowerValidationPoints | **Not consumed** | Same. |
| ServiceWaterHeaterValidationPoints | **Not consumed** | Same. |

### 2.5 Solar Thermal

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| SolarThermalCollector | SOLAR-FRACTION on DW-HEATER | DOE-2.2 uses a simplified solar savings fraction, not a collector model. |
| SolarEnergyFraction | SOLAR-FRACTION | **Primary for DOE-2.2.** Direct mapping. |
| CollectorArea / Tilt / Azimuth / FRTA / FRUL | **Not consumed** | DOE-2.2 does not model collectors; use SolarEnergyFraction. |

### 2.6 Desuperheater Heat Recovery

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| DesuperheaterHeatRecovery | **Not consumed** | DOE-2.2 does not model desuperheater DHW recovery. Informational. |
| DesuperheaterEfficiency | **Not consumed** | |

### 2.7 Distribution

| gbXML Element | DOE-2.2 Object/Keyword | Notes |
|---|---|---|
| ServiceWaterDistribution | CIRCULATION-LOOP (TYPE=DHW) | One loop per system. |
| DesignSupplyTemperature | LOOP-DESIGN-DT or setpoint | Supply temperature. |
| DesignInletTemperature | DHW-INLET-T | Cold water mains temp. |
| EnteringWaterTempScheduleIdRef | DHW-INLET-T schedule | Monthly or hourly. |
| RecirculationPumpPower | PUMP on CIRCULATION-LOOP | Motor watts. |
| RecirculationControlType | Pump availability schedule | Timer/demand mapped to scheduling. |
| LoopHeatLoss | **Primary for DOE-2.2** | Aggregate loop standby loss (Btu/h). |
| RecirculationDesignFlowRate | **Primary for DOE-2.2** | Loop pump sizing flow (GPM). |
| SimplifiedDistributionType | **Useful for eQUEST Wizard** | System classification for wizard default selection. |
| DistributionSystemMultiplier | **Informational** | Useful for wizard parametrics. |
| ServiceWaterPiping | **Not consumed** | DOE-2.2 uses LoopHeatLoss. |
| SegmentRole / BranchWasteVolume | **Not consumed** | |
| DrainHeatRecoveryEfficiency | **Not modeled** | DOE-2.2 does not model DWHR. |
| AreRecircPumpMotorsInThermalZone | **Not consumed** | |

## 3. Exporter Notes

- DOE-2.2 supports one DW-HEATER per CIRCULATION-LOOP. Multiple schema WaterHeaters must be combined (sum capacities, weighted-average efficiencies, sum tank volumes).
- SystemSeasonalEfficiency is preferred: HIR = 1/SystemSeasonalEfficiency.
- LoopHeatLoss and RecirculationDesignFlowRate are preferred distribution inputs. If absent, compute from piping geometry.
- Swing tanks are combined into primary DW-HEATER TANK-VOLUME; staging not representable.
- Solar thermal maps to SOLAR-FRACTION keyword; use SolarEnergyFraction directly when present. If absent, the exporter must derive solar fraction from collector parameters (non-trivial; f-chart or similar).
- Desuperheater, DWHR, cold water, and irrigation loads are not representable in DOE-2.2.
