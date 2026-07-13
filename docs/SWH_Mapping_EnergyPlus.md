# gbXML SWH Extension — EnergyPlus Mapping Guide

**Companion to:** gbXML Service Water Heating Extension Schema v0.3 (merged)
**Engine:** EnergyPlus (v24.2+, via OpenStudio SDK or direct IDF)
**Date:** June 2026

---

## 1. Overview

EnergyPlus models service water heating through a plant-loop architecture with explicit equipment objects, tank thermodynamics (mixed or stratified), and schedule-driven demand. The gbXML SWH extension maps naturally to this architecture. EnergyPlus is a **component-level, geometry-aware, hourly-schedule** engine for SWH. It consumes the detailed data paths and generally ignores the aggregate/simplified paths.

## 2. Element Mapping

### 2.1 Demand Side

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| ServiceWaterLoad | WaterUse:Equipment | One object per load. Peak flow from WaterUseValue; schedule from WaterUseScheduleIdRef. |
| WaterUseValue (VolumeFlowRate) | Peak Flow Rate (m3/s) | Convert gpm to m3/s. |
| WaterUseScheduleIdRef | Flow Rate Fraction Schedule Name | Fractional 0–1 schedule. |
| HotWaterUseFraction | (derived) | Fraction of WaterUseValue that is hot water. |
| HotWaterSupplyTemperature | Target Temperature Schedule | Can be constant or scheduled. |
| InletColdWaterTemperature | (on WaterUse:Connections) | Cold water supply temp. |
| DrainHeatRecoveryEfficiency | WaterUse:Equipment DWHR fields | Effectiveness, counterflow/crossflow. |
| DrainHeatRecoveryType | Drain Water Heat Exchanger Type | Vertical → CounterFlow; Horizontal → CrossFlow. |
| ColdWaterLoad | WaterUse:Equipment (cold only) | Hot fraction = 0; represents non-heated water use. |
| ColdWaterLoadType | (informational) | Irrigation, CoolingTower, Flushing, etc. |
| DrawProfileIdRef | **Not consumed** | EnergyPlus uses hourly schedules. |
| ZoneAggregateDHWDemand | **Not consumed** | EnergyPlus sums per-Space loads. |
| ServiceWaterZone | No direct equivalent | Organizational; exporter assigns loads to WaterUse:Connections on the plant loop. |

### 2.2 Supply Side (Equipment)

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| WaterHeater (Storage) | WaterHeater:Mixed or WaterHeater:Stratified | Choice depends on TankType (Mixed/Stratified). |
| WaterHeater (Instantaneous) | WaterHeater:Mixed | Tank volume = 0. |
| WaterHeater (HeatPump) | WaterHeater:HeatPump:WrappedCondenser or PumpedCondenser | Residential vs commercial. |
| WaterHeater (Indirect) | WaterHeater:Mixed with source-side connection | SourceHydronicLoopIdRef maps to source plant loop. |
| WaterHeater (HotWaterSupplyBoiler) | Boiler:HotWater on DHW loop | Dedicated DHW boiler. |
| WaterHeater (CombinationBoiler) | Boiler:HotWater serving both space and DHW loops | Dual-service. |
| FuelSource | Heater Fuel Type | NaturalGas, Electricity, Propane, FuelOilNo1, FuelOilNo2, DistrictHeating. |
| InputCapacity | Heater Maximum Capacity | Convert Btu/h to W. |
| RatedCapacity | Heater Maximum Capacity (output) | Heating output capacity. |
| ThermalEfficiency | Heater Thermal Efficiency | Dimensionless. |
| UniformEnergyFactor | (derived) | UEF not directly used; map to thermal efficiency + standby. |
| EnergyFactor | (derived) | Legacy; convert to Et + standby. |
| RecoveryEfficiency | Recovery Efficiency | For gas storage heaters. |
| StandbyLossFraction | Off-Cycle Loss Fraction to Ambient | Dimensionless. |
| SetpointTemperature | Setpoint Temperature Schedule | Constant or scheduled. |
| CutInTemperatureDelta | Deadband Temperature Difference | Degrees. |
| CompressorZoneIdRef | Inlet Air Zone Name | Zone for HPWH air source. |
| SourceAirConfiguration | Inlet Air Configuration | ZoneAir→"Zone Air Only", OutdoorAir→"Outdoor Air Only", DuctedMixed→"Zone and Outdoor Air". |
| DuctedOutdoorAirFraction | Inlet Air Mixer Schedule Value | Constant or scheduled fraction. |
| HPWHProductModelDesignation | **Informational** | EnergyPlus uses explicit performance curves. |
| SourceHydronicLoopIdRef | Source Side Inlet/Outlet Node | Connects indirect heater to boiler/solar loop. |
| BackupHeaterFuelSource | (on HPWH) Supplemental Heater Fuel | Backup element fuel type. |
| BackupInputCapacity | (on HPWH) Supplemental Heater Capacity | Backup element capacity. |
| SystemSeasonalEfficiency | **Not consumed** | EnergyPlus models each component individually. |

### 2.3 Storage Tanks

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| StorageTank (Primary) | Part of WaterHeater:* | Integral to the heater object. |
| StorageTank (Swing) | WaterHeater:Mixed (storage-only) | No heating source; callbacks via plant-loop setpoint manager. |
| StorageTank (Preheat) | WaterHeater:Mixed upstream of primary | Solar or DWHR preheat tank on supply side. |
| TankRole | Determines object configuration | Primary = integral; Swing = separate storage-only; Preheat = upstream. |
| LinkedPrimaryTankIdRef | Plant loop topology | Swing/preheat placed relative to primary on same loop. |
| SwingTankMaintenanceSetpoint | Setpoint Temperature Schedule | On the swing tank's WaterHeater:Mixed object. |
| TankType (Mixed) | WaterHeater:Mixed | Single-node tank model. |
| TankType (Stratified) | WaterHeater:Stratified | Multi-node tank model. |
| StorageVolume | Tank Volume | Convert gallons to m3. |
| StandbyLossCoefficient | Off-Cycle Loss Coefficient to Ambient Temperature | UA value (W/K). |
| InsulationRValue | (derived) | Compute UA from R-value and tank geometry. |
| TankZoneIdRef | Ambient Temperature Zone Name | Zone providing ambient conditions. |
| NumberOfNodes | Number of Nodes (Stratified) | Number of stratification nodes. |
| HeightToDiameterRatio | Tank Shape / Height | Determines tank geometry. |

### 2.4 HPWH Performance

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| HPWHCapacityValidationPoints | Curve:Biquadratic (heating capacity) | Generate biquadratic curve from validation points as f(Twb, Tinlet). |
| HPWHPowerValidationPoints | Curve:Biquadratic (compressor power) | Generate biquadratic curve from validation points as f(Twb, Tinlet). |
| ServiceWaterHeaterValidationPoints | Performance curve generation | General performance validation data. |
| AmbientDryBulbTemperature | Independent variable 1 | Evaporator air temp. |
| InletWaterTemperature | Independent variable 2 | Entering water temp. |
| HeatingCapacity / InputPower | Dependent variable | Output at operating point. |

### 2.5 Solar Thermal

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| SolarThermalCollector | SolarCollector:FlatPlate:Water | Solar DHW preheat. |
| CollectorArea | Gross Area | m2. |
| CollectorTilt | Tilt | Degrees from horizontal. |
| CollectorAzimuth | Azimuth | Degrees from north. |
| CollectorType | Collector Performance Type | Flat plate, evacuated tube, etc. |
| FRTA | FR(tau-alpha) | Optical efficiency (intercept of collector curve). |
| FRUL | FR-UL | Heat loss coefficient (slope of collector curve). |
| SolarEnergyFraction | (result, not input) | Informational; EnergyPlus calculates solar fraction. |
| LinkedStorageTankIdRef | Storage tank node on solar loop | Solar preheat tank connection. |

### 2.6 Desuperheater Heat Recovery

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| DesuperheaterHeatRecovery | Coil:WaterHeating:Desuperheater | Recovers waste heat from refrigeration/AC condenser. |
| DesuperheaterEfficiency | Heat Reclaim Recovery Efficiency | Fraction of condenser heat recovered. |
| MaxHeatRecoveryCapacity | (sizing limit) | Maximum recovery rate. |
| LinkedHVACEquipmentId | Heating Source Object Name | DX coil or refrigeration condenser providing waste heat. |

### 2.7 Distribution

| gbXML Element | EnergyPlus Object | Notes |
|---|---|---|
| ServiceWaterDistribution | WaterUse:Connections + PlantLoop | Organizes demand connections on the loop. |
| DesignSupplyTemperature | PlantLoop setpoint | Via SetpointManager:Scheduled. |
| DesignInletTemperature | Site:WaterMainsTemperature | Or schedule. |
| EnteringWaterTempScheduleIdRef | Water Mains Temperature Schedule | Monthly or hourly mains temp. |
| RecirculationPumpPower | Pump:VariableSpeed or ConstantSpeed | On the DHW plant loop. |
| RecirculationControlType | Pump availability schedule | Timer/demand mapped to pump scheduling. |
| ServiceWaterPiping | Pipe:Indoor / Pipe:Outdoor | Geometric: diameter, length, insulation. |
| AreRecircPumpMotorsInThermalZone | (pump zone assignment) | Whether pump heat goes to zone. |
| SegmentRole | **Not consumed** | EnergyPlus uses geometric pipe objects. |
| BranchWasteVolume | **Not consumed** | Per-draw branch waste not modeled. |
| SimplifiedDistributionType | **Not consumed** | EnergyPlus uses piping geometry. |
| LoopHeatLoss | **Not consumed** | Computed from pipe geometry. |
| RecirculationDesignFlowRate | **Not consumed** | Determined by pump sizing. |

## 3. Exporter Notes

- Each ServiceWaterLoad becomes a separate WaterUse:Equipment on a shared WaterUse:Connections object.
- Multiple WaterHeater objects can be placed on the same PlantLoop branch for staged operation.
- Swing tanks are modeled as WaterHeater:Mixed with zero heating capacity, placed downstream on the loop.
- Preheat tanks (solar, DWHR) are placed upstream on the supply side of the plant loop.
- HPWH validation points generate EnergyPlus biquadratic curves for COP and capacity as functions of ambient and inlet water temperature.
- Solar thermal collectors connect via a separate solar loop to the preheat tank, with a SolarCollector:FlatPlate:Water object and pump.
- Desuperheater recovery connects to a WaterHeater:Mixed (preheat or primary) with the Coil:WaterHeating:Desuperheater referencing the source DX/refrigeration equipment.
- All simplified and aggregate fields are informational for EnergyPlus; the exporter always uses detailed component and geometric data.
