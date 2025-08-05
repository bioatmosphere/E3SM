# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

E3SM (Energy Exascale Earth System Model) is a state-of-the-art fully coupled Earth system model designed for Department of Energy mission needs and leadership computing facilities. The repository contains multiple climate model components and comprehensive tooling for building, configuring, and running simulations.

## Key Commands

### Model Setup and Execution
```bash
# Create a new case using CIME
./cime/scripts/create_newcase --case <case_name> --compset <compset> --res <resolution> --machine <machine>

# Setup the case 
cd <case_name>
./case.setup

# Build the model
./case.build

# Submit the simulation
./case.submit
```

### Using the Template Script
```bash
# Use the provided template for Water Cycle simulations
cp run_e3sm.template.sh my_run_script.sh
# Edit configuration variables in the script
bash my_run_script.sh
```

### Component-Specific Operations

#### SCREAM/EAMxx (Next-generation atmosphere model)
```bash
# Test all SCREAM functionality
./components/eamxx/scripts/test-all-scream

# Query SCREAM configuration
./components/eamxx/scripts/query-scream

# Modify atmosphere parameters
./components/eamxx/scripts/atmchange <parameter> <value>

# Gather performance data across machines
./components/eamxx/scripts/gather-all-data
```

#### ELM (Land Model)
```bash
# Build ELM namelist
./components/elm/bld/build-namelist

# Query default namelist values
./components/elm/bld/queryDefaultNamelist.pl
```

### Development and Testing
```bash
# Run component-specific tests (varies by component)
# Example for SCREAM:
cd components/eamxx
./scripts/test-all-scream --test-suite <suite_name>

# Check code formatting and standards
# (Commands vary by component - see individual README files)
```

## Architecture Overview

E3SM follows a modular component-based architecture with the following main components:

### Core Components
- **EAM/SCREAM**: Atmosphere model (traditional EAM vs next-gen SCREAM)
- **ELM**: Energy Exascale Land Model for terrestrial processes
- **MPAS-Ocean**: Unstructured mesh ocean model
- **MPAS-Seaice**: Sea ice component using unstructured meshes
- **CICE**: Alternative sea ice model
- **MOSART**: River routing model
- **MPAS-Albany-Landice**: Ice sheet dynamics
- **WW3**: Wave model component

### Framework and Infrastructure
- **CIME**: Common Infrastructure for Modeling Earth (build system, case management)
- **Driver**: Coupler components (MCT-based and MOAB-based options)
- **Share**: Common utilities and shared code
- **Data Components**: Prescribed data inputs (datm, docn, dlnd, etc.)
- **Stub Components**: Minimal placeholder components

### Build System
E3SM uses CMake as the primary build system with CIME providing the orchestration layer. Each component has its own CMakeLists.txt and build configuration.

## Development Workflow

### Code Organization
- Component source code: `components/<component>/src/`
- Build configurations: `components/<component>/cime_config/`
- Testing: `components/<component>/test/` or `components/<component>/tests/`
- Tools: `components/<component>/tools/`

### Key Configuration Files
- **XML Files**: Component configurations, machine definitions, compsets
  - `cime_config/machines/config_machines.xml`: Machine definitions
  - `cime_config/config_compsets.xml`: Component set definitions
  - `cime_config/config_grids.xml`: Grid definitions
- **Namelist Files**: Runtime parameters for each component
- **Build Scripts**: `buildlib_cmake` and `buildnml` for each component

### Testing Approach
Each component has its own test suite:
- SCREAM: `test-all-scream` script with comprehensive testing
- ELM: Test scripts in `components/elm/test/tools/`  
- MPAS components: Individual makefiles and test configurations
- System-level tests defined in `cime_config/testdefs/`

### Machine Configuration
E3SM supports numerous HPC systems with machine-specific configurations:
- Compiler settings: `cime_config/machines/cmake_macros/`
- Machine definitions: `cime_config/machines/config_machines.xml`
- Batch system templates: `cime_config/machines/template.*`

## Common Patterns

### Fortran 90/95/2003 Standards
Most components are written in Fortran with some C/C++ utilities. Follow existing code style within each component.

### CMake Integration
Components integrate with the overall build system through:
```cmake
# Component-level CMakeLists.txt patterns
add_library(<component> <sources>)
target_include_directories(<component> ...)
```

### Namelist Handling
Each component uses XML-based namelist definition and default files:
- `namelist_definition.xml`: Parameter definitions
- `namelist_defaults_<component>.xml`: Default values

### CIME Integration
Components integrate with CIME through:
- `buildlib_cmake`: CMake build script
- `buildnml`: Namelist generation script  
- `config_component.xml`: Component metadata

## Tips for Development

- Always test changes with relevant component test suites before submitting
- Use the template script (`run_e3sm.template.sh`) as a starting point for new simulations
- Consult component-specific README files for detailed development guidance
- SCREAM has the most comprehensive modern tooling - reference it for best practices
- Machine-specific issues often require updates to cmake macro files
- Component coupling happens through the driver - understand field exchanges when modifying interfaces

## Important Notes

- E3SM requires significant computational resources - most development should be done on supported HPC systems
- Different components have different maturity levels and development practices
- SCREAM represents the next-generation development approach with modern C++ and comprehensive testing
- Always coordinate with component teams for major changes to core algorithms or interfaces