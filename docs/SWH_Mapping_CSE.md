# gbXML SWH Extension — CSE Mapping Guide

**Companion to:** gbXML Service Water Heating Extension Schema v0.3 (merged)
**Engine:** California Simulation Engine (CSE)
**Date:** June 2026

---

## 1. Overview

CSE models domestic hot water through a discrete-draw event engine with explicit loop topology. CSE simulates individual draw events (start, duration, flow, temperature) and separates recirculation piping into mains (standby-loss-driven) and branches (per-draw-waste-driven). CSE also uses manufacturer product libraries for HPWH performance rather than generic curves. CSE is a **draw-event, loop-topology, product-library** engine.

## 2. Element Mapping

### 2.1 Demand Side

| gbXML Element | CSE Object | Notes |
|---|---|---|
| ServiceWaterDrawProfile | DHWDAYUSE | Named collection of draws for one day type. Top-level, reusable. |
| ServiceWaterDraw | DHWUSE | Individual draw event within a DHWDAYUSE. |
| EndUse | wuHWEndUse | Shower, Bath, Faucet, CWashr, DWashr. |
| StartHour | wuStart | Decimal hour 0–24. |
| DurationMinutes | wuDuration | Minutes. |
| FlowRate | wuFlow | GPM (mixed water flow at fixture). |
| Volume | wuVol | Gallons (alternative to flow × duration). |
| MixedDeliveryTemperature | wuTemp | Target mixed water temp at fixture. |
| HotWaterFraction | wuHotF | Alternative to MixedDeliveryTemperature. |
| EventId | wuEventID | Groups sub-draws for DWHR simultaneity. |
| DrawProfileIdRef | (on ServiceWaterLoad) | Links a Space's load to its DHWDAYUSE. **Primary demand path.** |
| WaterUseValue + Schedule | **Fallback** | Only if no DrawProfileIdRef. |
| ZoneAggregateDHWDemand | **Not consumed** | CSE sums per-Space draws. |
| ColdWaterLoad | **Not consumed** | CSE does not model cold water loads. |

### 2.2 Supply Side (Equipment)

| gbXML Element | CSE Object | Notes |
|---|---|---|
| WaterHeater | DHWHEATER | One per physical heater. |
| WaterHeaterType | whType | Storage→SmallStorage/LargeStorage, HeatPump→SmallInstantaneous (with HP), CentralHeatPumpWaterHeater→LargeStorage (with HP). |
| FuelSource | whFuel | Elec, Gas, etc. |
| InputCapacity | whParElec / whCap | Input power. |
| RatedCapacity | whCapR | Rated heating output. |
| SetpointTemperature | whTSetpoint | Tank setpoint. |
| CutInTemperatureDelta | (deadband) | Degrees below setpoint triggering heat call. |
| UniformEnergyFactor | whEF (mapped from UEF) | UEF-to-EF conversion may be needed. |
| EnergyFactor | whEF | Direct mapping for legacy equipment. |
| RecoveryEfficiency | whRE | For gas storage heaters. |
| HPWHProductModelDesignation | (product library selection) | **Primary for CSE.** Selects built-in performance data. Overrides generic curves. |
| SourceAirConfiguration | wsSrc | ZoneAir→Zone, OutdoorAir→Outdoor, DuctedMixed→Zone (with duct modifiers). |
| DuctedOutdoorAirFraction | (duct outdoor air mix) | Fraction of source air from outdoors. |
| SourceAirTemperatureScheduleIdRef | (source temp override) | Explicit evaporator inlet temp. |
| CompressorZoneIdRef | (HPWH zone assignment) | Zone providing source air. |
| BackupHeaterFuelSource | (backup fuel) | Backup element fuel. |
| BackupInputCapacity | (backup capacity) | Backup element input power. |
| SourceHydronicLoopIdRef | **Not consumed** | CSE does not model indirect heaters via hydronic loops. |
| SystemSeasonalEfficiency | **Not consumed** | CSE models components individually. |

### 2.3 Storage Tanks

| gbXML Element | CSE Object | Notes |
|---|---|---|
| StorageTank (Primary) | Part of DHWHEATER | Integral tank parameters on the heater. |
| StorageTank (Swing) | Separate DHWHEATER (whType=StorageTank) | Storage-only heater with temp-maintenance controls. |
| StorageTank (Preheat) | Separate DHWHEATER upstream | Solar/DWHR preheat tank. |
| TankRole | Determines DHWHEATER configuration | Primary = integral; Swing = storage-only staging node. |
| LinkedPrimaryTankIdRef | DHWHEATER staging linkage | Swing calls primary heater for heat. |
| SwingTankMaintenanceSetpoint | whTSetpoint (on swing DHWHEATER) | Triggers callback to primary. |
| StorageVolume | whVol | Tank volume in gallons. |
| StandbyLossCoefficient | whUA | Tank UA (Btu/h-°F). |
| TankZoneIdRef | (zone ambient assignment) | Tank ambient conditions. |
| TankType | **Informational** | CSE uses its own tank model. |

### 2.4 HPWH Performance

| gbXML Element | CSE Object | Notes |
|---|---|---|
| HPWHCapacityValidationPoints | **Informational** | CSE uses Ecotope product model when HPWHProductModelDesignation is present. Falls back to generic model from UEF/EF if no product match. |
| HPWHPowerValidationPoints | **Informational** | Same as above. |
| ServiceWaterHeaterValidationPoints | **Informational** | General performance data; CSE uses built-in correlations. |

### 2.5 Solar Thermal

| gbXML Element | CSE Object | Notes |
|---|---|---|
| SolarThermalCollector | DHWSOLARSYS | Solar DHW preheat system. |
| CollectorArea | (collector area) | m² or ft². |
| CollectorTilt | (collector tilt) | Degrees from horizontal. |
| CollectorAzimuth | (collector azimuth) | Degrees from north. |
| FRTA | (optical efficiency) | FR(τα) intercept. |
| FRUL | (heat loss coefficient) | FR-UL slope. |
| SolarEnergyFraction | wsSSF | Solar savings fraction (can be input or result). |
| LinkedStorageTankIdRef | (preheat tank linkage) | Associates collector with preheat tank. |

### 2.6 Desuperheater Heat Recovery

| gbXML Element | CSE Object | Notes |
|---|---|---|
| DesuperheaterHeatRecovery | **Limited support** | CSE has limited desuperheater modeling; map to a supplemental heat source or preheat contribution. |
| DesuperheaterEfficiency | (recovery fraction) | Applied as preheat. |

### 2.7 Distribution

| gbXML Element | CSE Object | Notes |
|---|---|---|
| ServiceWaterDistribution | DHWSYS / DHWLOOP | System and loop configuration. |
| ServiceWaterPiping (SupplyMain) | DHWLOOPSEG (wgTy=supply) | Standby loss = f(flow, UA, ambient). |
| ServiceWaterPiping (ReturnMain) | DHWLOOPSEG (wgTy=return) | Return standby loss. |
| ServiceWaterPiping (Branch/Twig) | DHWLOOPBRANCH | Per-draw distribution waste. |
| SegmentRole | wgTy / branch classification | **Primary for CSE.** Selects loss model. |
| RecirculationFlowRate | (segment recirc flow) | Flow while no draw for this segment. |
| FractionOfLoopFlowWhileNoDraw | wgFNoDraw | Fraction of total loop flow. |
| BranchWasteVolume | (branch waste volume) | Cooled water reheated each draw start. |
| BranchFlowRate | wbFlow | Flow during active draw. |
| SegmentMultiplier | wgMult / wbMult | N identical parallel segments. |
| SimplifiedDistributionType | wsDSM (via lookup) | **Primary for non-recirc residential.** System type → default multiplier. |
| DistributionSystemMultiplier | wsDSM (direct) | Overrides default. |
| AreRecircPumpMotorsInThermalZone | (pump heat assignment) | Whether pump heat goes to zone or pipe fluid. |
| LoopHeatLoss | **Not consumed** | CSE computes from segment geometry. |
| RecirculationDesignFlowRate | **Not consumed** | CSE sums per-segment flows. |

## 3. Exporter Notes

- The draw-event path (DrawProfileIdRef → ServiceWaterDrawProfile → ServiceWaterDraw) is the primary demand path. Peak × schedule is fallback only.
- For non-recirc residential (no DHWLOOP), use SimplifiedDistributionType + DistributionSystemMultiplier instead of piping geometry.
- HPWHProductModelDesignation is the preferred HPWH performance path; generic UEF/EF + validation points are fallback.
- Swing tanks become separate DHWHEATER objects with zero heating capacity and staging linkage.
- SourceAirConfiguration maps directly to wsSrc; DuctedMixed requires duct-loss modifiers from DuctedOutdoorAirFraction.
- Cold water loads (irrigation, cooling tower, flushing) are not modeled in CSE.
