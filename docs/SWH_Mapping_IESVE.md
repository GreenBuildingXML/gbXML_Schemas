# gbXML SWH Extension — IES-VE Mapping Guide

**Companion to:** gbXML Service Water Heating Extension Schema v0.3 (merged)
**Engine:** IES Virtual Environment (Apache Systems + ApacheHVAC)
**Date:** June 2026

---

## 1. Overview

IES-VE models domestic hot water through two subsystems: Apache Systems (loads, tanks, solar, generator efficiency) and ApacheHVAC (plant loops with Generic Heat Sources, recirculation, heat pumps, thermal storage). Demand is defined per-room or per-zone.

**Critical note:** IES-VE's gbXML importer reads geometry only. SWH data must be configured programmatically via the IES-VE Python API (iesve module / VEScripts). IES-VE is a **zone-aggregate, seasonal-efficiency, plant-loop** engine.

## 2. Element Mapping

### 2.1 Demand Side

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| ZoneAggregateDHWDemand | DHW Zones dialog → Zone demand | **Primary.** Zone-level demand entered directly via VEScript. |
| ZoneAggregateDHWDemandScheduleIdRef | DHW Zones dialog → Modulating profile | Linked profile for time variation. |
| ServiceWaterLoad (per-Space) | Building Template Manager → Space Conditions → DHW | **Fallback.** Room-level peak demand with modulating profile. |
| WaterUseValue | Peak demand (kW or L/s) | Unit conversion needed. |
| WaterUseScheduleIdRef | Modulating profile | Fractional 0–1 profile. |
| HotWaterUseFraction | (fraction applied to demand) | Fraction of total that is hot water. |
| ColdWaterLoad | **Not consumed** | IES-VE is energy-focused; cold water not modeled. |
| DrawProfileIdRef | **Not consumed** | IES-VE uses hourly profiles. |
| ServiceWaterZone | DHW Zone grouping | Maps directly to IES-VE DHW Zone. |

### 2.2 Supply Side (Equipment)

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| SystemSeasonalEfficiency | Apache Systems → Generator Seasonal Efficiency | **Primary.** Single annualized value. |
| WaterHeater (basic params) | ApacheHVAC → Generic Heat Source (GHS) | Type, fuel, capacity. |
| WaterHeaterType=HeatPump | GHS → ASHP tab | Air-source heat pump primary with backup. |
| WaterHeaterType=Indirect | GHS → source from space heating boiler | Indirect via hydronic loop. |
| WaterHeaterType=HotWaterSupplyBoiler | GHS → dedicated boiler | Dedicated DHW boiler in ApacheHVAC. |
| FuelSource | GHS → Fuel type | Electricity, gas, etc. |
| InputCapacity / RatedCapacity | GHS → Heating capacity | Output in kW. |
| SetpointTemperature | GHS → Setpoint | DHW supply setpoint. |
| CompressorZoneIdRef | GHS → ASHP → Inlet Air Zone | Zone providing evaporator air. |
| SourceAirConfiguration | GHS → ASHP → Air source type | ZoneAir, OutdoorAir, DuctedMixed. |
| DuctedOutdoorAirFraction | GHS → ASHP → Outdoor fraction | For ducted configurations. |
| HPWHProductModelDesignation | **Informational** | IES-VE uses COP-vs-temperature internal model. |
| SourceHydronicLoopIdRef | GHS → source loop connection | For indirect heaters connected to boiler loop. |
| BackupHeaterFuelSource | GHS → backup heating element | Backup within GHS dialog. |
| BackupInputCapacity | GHS → backup capacity | Backup element sizing. |

### 2.3 Storage Tanks

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| StorageTank (Primary) | Apache Systems → DHW tank or ApacheHVAC → DHW tank | Volume, UA, ambient zone. |
| StorageTank (Swing) | ApacheHVAC → Additional DHW storage | Additional storage with temperature maintenance. |
| StorageTank (Preheat) | Apache Systems → Solar preheat tank | Upstream of primary heater. |
| StorageVolume | Tank volume (liters) | Convert gallons to liters. |
| StandbyLossCoefficient | Tank UA (W/K) | Convert Btu/h-°F to W/K. |
| TankZoneIdRef | Tank ambient zone | Determines ambient temp for standby losses. |
| TankRole | Configuration guidance | Determines VEScript tank setup. |
| SwingTankMaintenanceSetpoint | Temperature maintenance setpoint | Triggers heat call to primary. |

### 2.4 HPWH Performance

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| HPWHCapacityValidationPoints | **Informational** | IES-VE uses COP-vs-temperature internal curves. |
| HPWHPowerValidationPoints | **Informational** | Same. |
| ServiceWaterHeaterValidationPoints | **Informational** | Same. |

### 2.5 Solar Thermal

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| SolarThermalCollector | Apache Systems → Solar Hot Water panel | IES-VE has native solar DHW modeling. |
| CollectorArea | Panel area | m². |
| CollectorTilt | Panel tilt | Degrees from horizontal. |
| CollectorAzimuth | Panel azimuth | Degrees from north. |
| CollectorType | Panel type | Flat plate, evacuated tube. |
| FRTA | Optical efficiency | FR(τα) intercept. |
| FRUL | Heat loss coefficient | FR-UL slope. |
| SolarEnergyFraction | **Informational** | IES-VE calculates solar fraction from collector model. |
| LinkedStorageTankIdRef | Solar preheat tank connection | Links collector to preheat tank in Apache Systems. |

### 2.6 Desuperheater / Condenser Heat Recovery

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| DesuperheaterHeatRecovery | ApacheHVAC → Condenser heat recovery option | IES-VE supports condenser heat recovery to DHW. |
| DesuperheaterEfficiency | Recovery efficiency | Fraction of condenser heat recovered. |
| MaxHeatRecoveryCapacity | Maximum recovery capacity | Sizing limit. |
| LinkedHVACEquipmentId | Source refrigeration/AC equipment | Links to the condenser providing waste heat. |

### 2.7 Distribution

| gbXML Element | IES-VE Location | Notes |
|---|---|---|
| ServiceWaterDistribution | Apache Systems → DHW loop / ApacheHVAC plant loop | System-level config. |
| DesignSupplyTemperature | Loop supply setpoint | Note: IES-VE defaults to 50 K rise from mains; explicit temps override. |
| DesignInletTemperature | Cold water inlet temperature | Mains water temp. |
| EnteringWaterTempScheduleIdRef | Monthly mains temp profile | Overrides IES-VE default. |
| RecirculationPumpPower | ApacheHVAC → Secondary circulation pump | Pump motor watts. |
| RecirculationControlType | Pump scheduling | Timer, demand, continuous. |
| SimplifiedDistributionType | **Informational** | IES-VE uses its own simplified recirc model. |
| LoopHeatLoss | Apache Systems → Recirc loss | Supplements IES-VE internal calculation. |
| DrainHeatRecoveryEfficiency | ApacheHVAC → DWHR | Drain waste heat recovery effectiveness. |
| DrainHeatRecoveryType | DWHR device type | Vertical, horizontal. |
| ServiceWaterPiping | **Not consumed** | IES-VE uses simplified recirc loss model. |
| SegmentRole / BranchWasteVolume | **Not consumed** | |
| AreRecircPumpMotorsInThermalZone | **Informational** | IES-VE handles pump heat internally. |

## 3. Exporter (VEScript) Implementation Notes

- Workflow: (1) import geometry via gbXML, (2) parse SWH data, (3) configure Apache Systems and ApacheHVAC programmatically via iesve Python API.
- ZoneAggregateDHWDemand is preferred: write to DHW Zone Properties in "zone-entered demand" mode.
- SystemSeasonalEfficiency maps to Apache Systems generator efficiency.
- Solar thermal collectors map to Apache Systems Solar Hot Water panel configuration; IES-VE computes solar fraction internally from collector parameters.
- Desuperheater/condenser recovery maps to ApacheHVAC condenser heat recovery options.
- ServiceWaterHeatingSpaceType (ASHRAE 90.1) may need a mapping table for NCM activity types in UK/EU compliance workflows.
- IES-VE's default 50 K temperature rise should be overridden with explicit DesignSupplyTemperature and DesignInletTemperature.
