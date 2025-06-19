# AIR_POLLUTION_FLOW_USING_OPENFOAM

# OpenFOAM Air Pollution Flow Simulation Around Tall Buildings

![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColorscription

This repository contains an OpenFOAM-based computational fluid dynamics (CFD) simulation for modeling air pollution flow around tall buildings in urban environments . The project implements a passive scalar transport equation to simulate pollutant dispersion in complex building geometries, providing valuable insights into air quality assessment and urban planning .

## Table of Contents

- [Project Description](#project-description)
- [Prerequisites](#prerequisites)
- [Installation Guide](#installation-guide)
  - [Setting up WSL](#setting-up-wsl)
  - [Installing OpenFOAM](#installing-openfoam)
  - [Verification](#verification)
- [Case Setup](#case-setup)
  - [Directory Structure](#directory-structure)
  - [Mesh Generation](#mesh-generation)
  - [Boundary Conditions](#boundary-conditions)
  - [Solver Configuration](#solver-configuration)
- [Running the Simulation](#running-the-simulation)
- [Post-Processing](#post-processing)
- [Troubleshooting](#troubleshooting)
- [Results and Visualization](#results-and-visualization)
- [License](#license)

## Prerequisites

- Windows 10/11 with WSL2 support 
- At least 8GB RAM and 20GB free disk space
  

## Installation Guide

### Setting up WSL

1. **Open Windows Command Prompt as Administrator**
   ```bash
   # Check if WSL is already installed
   wsl -l -v
   ```

2. **Install WSL with Ubuntu 22.04** 
   ```bash
   wsl --install -d Ubuntu-22.04
   ```

3. **Configure Ubuntu** 
   - When prompted, create a username and password
   - After installation, verify the installation:
   ```bash
   wsl -l -v
   ```

4. **Start WSL** 
   ```bash
   wsl
   ```

### Installing OpenFOAM

1. **Add OpenFOAM Repository** 
   ```bash
   curl https://dl.openfoam.com/add-debian-repo.sh | sudo bash
   sudo apt-get update
   ```

2. **Install OpenFOAM** 
   ```bash
   # Install latest version
   sudo apt-get install openfoam-default
   
   # Or install specific version (e.g., v2312)
   sudo apt-get install openfoam2312-default
   ```

3. **Set up Environment Variables** 
   ```bash
   # Add OpenFOAM function to bash aliases
   wget https://raw.githubusercontent.com/jakobhaervig/openfoam-installer/main/.bash_aliases -O - >> $HOME/.bash_aliases
   
   # Source bashrc file
   source $HOME/.bashrc
   ```

### Verification

Test the installation by running a simple tutorial :
```bash
# Create working directory
mkdir -p $HOME/OpenFOAM/$USER-run
cd $HOME/OpenFOAM/$USER-run

# Copy and run cavity tutorial
cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity/cavity ./
cd cavity
blockMesh
icoFoam
```

## Case Setup

### Directory Structure

The OpenFOAM case follows the standard structure :

```
airPollutionBuildings/
├── 0/                    # Initial and boundary conditions
│   ├── U                 # Velocity field
│   ├── p                 # Pressure field
│   ├── k                 # Turbulent kinetic energy
│   ├── epsilon           # Turbulent dissipation rate
│   └── pollutant         # Pollutant concentration
├── constant/             # Physical properties and mesh
│   ├── polyMesh/         # Mesh files
│   ├── transportProperties
│   └── turbulenceProperties
└── system/               # Solution control parameters
    ├── controlDict
    ├── fvSchemes
    ├── fvSolution
    ├── blockMeshDict
    └── snappyHexMeshDict
```

### Mesh Generation

1. **Create Background Mesh** 
   ```bash
   blockMesh
   ```

2. **Generate Complex Geometry with snappyHexMesh** 
   ```bash
   # Extract surface features
   surfaceFeatures
   
   # Generate mesh around buildings
   snappyHexMesh -overwrite
   ```

3. **Check Mesh Quality** 
   ```bash
   checkMesh
   ```

### Boundary Conditions

Configure boundary conditions for urban flow simulation :

**Inlet (Wind):**
```cpp
inlet
{
    type            atmBoundaryLayerInletVelocity;
    Uref            5.0;    // Reference wind speed
    Zref            10.0;   // Reference height
    zDir            (0 0 1);
    flowDir         (1 0 0);
    value           uniform (5 0 0);
}
```

**Pollutant Source:**
```cpp
pollutantSource
{
    type            fixedValue;
    value           uniform 1.0;  // Normalized concentration
}
```

### Solver Configuration

Use a modified solver for pollutant transport :
- Base solver: `buoyantBoussinesqPimpleFoam` or `simpleFoam`
- Additional passive scalar transport equation for pollutant 

## Running the Simulation

1. **Decompose Case for Parallel Processing** 
   ```bash
   decomposePar
   ```

2. **Run Simulation** 
   ```bash
   # Serial execution
   simpleFoam
   
   # Parallel execution
   mpirun -np 4 simpleFoam -parallel
   ```

3. **Reconstruct Results** 
   ```bash
   reconstructPar
   ```

## Post-Processing

### Using ParaView

1. **Create ParaView File** 
   ```bash
   touch case.foam
   ```

2. **Launch ParaView** 
   ```bash
   paraview case.foam
   ```

3. **Visualization Options** :
   - Contour plots for pollutant concentration
   - Vector plots for velocity fields
   - Streamlines for flow visualization
   - Slice plots through building regions

### Command Line Post-Processing

1. **Sample Data** 
   ```bash
   postProcess -func probes
   postProcess -func surfaces
   ```

2. **Extract Data** 
   ```bash
   foamToVTK
   ```

## Troubleshooting

### Common Issues and Solutions

1. **Segmentation Fault** 
   - **Cause**: Memory access outside bounds
   - **Solution**: Recompile with debug mode and check array bounds
   ```bash
   export WM_COMPILE_OPTION=Debug
   ```

2. **Syntax Errors** 
   - **Cause**: Missing semicolons or brackets in dictionary files
   - **Solution**: Check error messages carefully and verify file syntax
   ```bash
   # Error usually indicates file and line number
   ```

3. **Mesh Quality Issues** 
   - **Cause**: Poor mesh quality affecting convergence
   - **Solution**: Refine mesh settings in `snappyHexMeshDict`
   ```bash
   checkMesh
   ```

4. **Convergence Problems** 
   - **Cause**: Inappropriate solver settings or boundary conditions
   - **Solution**: Adjust relaxation factors and solver tolerances in `fvSolution`

5. **WSL Installation Errors** 
   - **Error**: `WslRegisterDistribution failed with error: 0x80370114`
   - **Solution**: Enable "Windows Hypervisor Platform" in Windows features

### Debugging Tips

1. **Check Log Files** 
   ```bash
   tail -f log.simpleFoam
   ```

2. **Validate Case Setup** 
   ```bash
   foamToVTK -check
   ```

3. **Monitor Residuals** 
   ```bash
   foamLog log.simpleFoam
   gnuplot residuals.plt
   ```

## Results and Visualization

The simulation provides insights into :
- Pollutant concentration distribution around buildings
- Flow patterns in urban canyon environments
- Wind velocity fields and turbulence characteristics
- Areas of high pollution accumulation

Typical results include:
- Concentration contours showing pollution dispersion
- Velocity vectors indicating flow patterns
- Streamlines revealing flow recirculation zones
- Time-series data for monitoring points

## Performance Considerations

- **Mesh Size**: Balance between accuracy and computational cost.
- **Parallel Processing**: Use domain decomposition for large cases.
- **Time Step**: Ensure Courant number < 1 for stability.
- **Turbulence Modeling**: k-ε model recommended for urban flows.

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details .

## Acknowledgments

- OpenFOAM Foundation for the open-source CFD software .
- Contributors to the OpenFOAM community tutorials.
- Research works on urban air pollution modeling.

## Contact

For questions or issues, please open an issue on GitHub or contact the maintainer.

---

**Last Updated**: June 2025
