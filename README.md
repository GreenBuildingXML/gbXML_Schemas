# gbXML Service Water Heating Extension

This repository contains current and previous gbXML schemas and a committee review package for the proposed **gbXML Service Water Heating (SWH) Extension v0.1**. The current published gbXML schema included here is version 8.01; the SWH extension is a draft proposal for review and is not yet part of the adopted gbXML standard.

The review package now includes the proposed extension schema, simulation-engine mapping documents, and example gbXML files showing how the extension can support multiple modeling workflows.

## Purpose

The Schema Review Committee is chartered to review, critique, and reach consensus on extensions that add SWH plant equipment, distribution networks, demand loads, draw profiles, and cold-water loads to gbXML. The intent is to make gbXML useful as an input data format across multiple energy simulation engines while ensuring that the proposal is technically sound, practically useful, standards-aligned, and non-breaking.

## Scope and committee goals

The committee's work includes:

- Reviewing the v0.1 draft coverage of water heaters, storage tanks, solar thermal systems, desuperheaters, piping, recirculation, simplified distribution multipliers, peak-and-schedule demand, discrete draw profiles, and cold-water loads.
- Evaluating the proposed data model against the requirements of the major simulation platforms represented by committee members.
- Establishing a common SWH vocabulary that works across building types and scales, from single-family homes to large commercial buildings.
- Verifying that every proposed element addresses at least one use case that existing gbXML elements cannot represent.
- Exercising the proposal through at least four use-case files, one for each major simulation platform represented on the committee.
- Identifying and resolving gaps, ambiguities, and redundancies.
- Recording accepted, rejected, and deferred proposals, including the rationale for each decision.
- Delivering a consensus draft suitable for submission to the gbXML Board of Directors for formal consideration and adoption.

Representative building use cases are expected to include single-family residential, multifamily, commercial office, and mixed-use projects.

## Design principles

The charter establishes five non-negotiable principles:

- **Non-breaking:** All additions must be optional. Existing gbXML elements, attributes, enumerations, and semantics must remain unchanged.
- **Multi-fidelity:** Optional parallel data paths must support both detailed component-level models and simplified aggregate models.
- **Standards-aligned:** Enumerations and data groups should align, where applicable, with ASHRAE Standard 229P, ASHRAE 90.1-2019 Section 7, and ASHRAE Standard 205.
- **Engine-neutral:** The schema must not reference or favor a simulation engine. Engine-specific guidance belongs in separate companion mapping documents.
- **Consensus-driven:** Accepted extensions require committee consensus; unresolved or contested items are documented and deferred rather than forced into the schema.

## Key technical questions

The review is intended to resolve several core modeling questions:

- Whether peak-and-schedule demand and discrete draw profiles should remain dual, parallel representations or whether one should be primary.
- Whether the `SupplyMain`, `ReturnMain`, `Branch`, and `Twig` piping roles cover the necessary distribution topologies.
- Whether `SimplifiedDistributionType` and `DistributionSystemMultiplier` adequately support residential models that do not warrant pipe-by-pipe geometry.
- Whether the tank-role and swing-tank staging model is sufficient for central multifamily heat-pump water-heater plants.
- Whether the heat-pump water-heater source-air model covers real-world configurations.
- Whether loop-level aggregate fields adequately serve engines that model at the loop or plant level.
- Whether `ServiceWaterHeatingSpaceType` belongs on `ServiceWaterZone`, `Space`, or both.

## Repository contents

### Proposed schema

- [`gbXML_ServiceWater_Extension_v0.1.xsd`](gbXML_ServiceWater_Extension_v0.1.xsd) - proposed Service Water Heating Extension v0.1 review draft.
- `GreenBuildingXML_Ver*.xsd` - current and historical gbXML schemas, including version 8.01 as the current baseline in this repository.

### Simulation-engine mapping documents

The files in [`docs/`](docs/) describe how engine-specific concepts consume or map to the engine-neutral extension. They are informative companion documents, not normative schema definitions.

- [`SWH_Mapping_CSE.md`](docs/SWH_Mapping_CSE.md)
- [`SWH_Mapping_DOE2_eQUEST.md`](docs/SWH_Mapping_DOE2_eQUEST.md)
- [`SWH_Mapping_EnergyPlus.md`](docs/SWH_Mapping_EnergyPlus.md)
- [`SWH_Mapping_IESVE.md`](docs/SWH_Mapping_IESVE.md)

### Example files

The files in [`examples/`](examples/) exercise the proposed elements for a representative 24-unit multifamily building with a central heat-pump water-heater plant and recirculation loop. Each example emphasizes the data path consumed by its target platform.

- [`gbXML_SWH_Example_CSE.xml`](examples/gbXML_SWH_Example_CSE.xml)
- [`gbXML_SWH_Example_DOE2_eQUEST.xml`](examples/gbXML_SWH_Example_DOE2_eQUEST.xml)
- [`gbXML_SWH_Example_EnergyPlus.xml`](examples/gbXML_SWH_Example_EnergyPlus.xml)
- [`gbXML_SWH_Example_IESVE.xml`](examples/gbXML_SWH_Example_IESVE.xml)
