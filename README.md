# AIR_POLLUTION_FLOW_USING_OPENFOAM

# OpenFOAM Air Pollution Flow Simulation Around Tall Buildings

![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColorscription

This repository contains an OpenFOAM-based computational fluid dynamics (CFD) simulation for modeling air pollution flow around tall buildings in urban environments [1]. The project implements a passive scalar transport equation to simulate pollutant dispersion in complex building geometries, providing valuable insights into air quality assessment and urban planning [2][3].

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

- Windows 10/11 with WSL2 support [4]
- At least 8GB RAM and 20GB free disk space
- Basic understanding of fluid dynamics and CFD concepts [5]

## Installation Guide

### Setting up WSL

1. **Open Windows Command Prompt as Administrator**
   ```bash
   # Check if WSL is already installed
   wsl -l -v
   ```

2. **Install WSL with Ubuntu 22.04** [4]
   ```bash
   wsl --install -d Ubuntu-22.04
   ```

3. **Configure Ubuntu** [6]
   - When prompted, create a username and password
   - After installation, verify the installation:
   ```bash
   wsl -l -v
   ```

4. **Start WSL** [4]
   ```bash
   wsl
   ```

### Installing OpenFOAM

1. **Add OpenFOAM Repository** [6]
   ```bash
   curl https://dl.openfoam.com/add-debian-repo.sh | sudo bash
   sudo apt-get update
   ```

2. **Install OpenFOAM** [6]
   ```bash
   # Install latest version
   sudo apt-get install openfoam-default
   
   # Or install specific version (e.g., v2312)
   sudo apt-get install openfoam2312-default
   ```

3. **Set up Environment Variables** [6]
   ```bash
   # Add OpenFOAM function to bash aliases
   wget https://raw.githubusercontent.com/jakobhaervig/openfoam-installer/main/.bash_aliases -O - >> $HOME/.bash_aliases
   
   # Source bashrc file
   source $HOME/.bashrc
   ```

### Verification

Test the installation by running a simple tutorial [6]:
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

The OpenFOAM case follows the standard structure [7]:

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

1. **Create Background Mesh** [8]
   ```bash
   blockMesh
   ```

2. **Generate Complex Geometry with snappyHexMesh** [8][9]
   ```bash
   # Extract surface features
   surfaceFeatures
   
   # Generate mesh around buildings
   snappyHexMesh -overwrite
   ```

3. **Check Mesh Quality** [8]
   ```bash
   checkMesh
   ```

### Boundary Conditions

Configure boundary conditions for urban flow simulation [10]:

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

Use a modified solver for pollutant transport [11][12]:
- Base solver: `buoyantBoussinesqPimpleFoam` or `simpleFoam`
- Additional passive scalar transport equation for pollutant [12]

## Running the Simulation

1. **Decompose Case for Parallel Processing** [6]
   ```bash
   decomposePar
   ```

2. **Run Simulation** [11]
   ```bash
   # Serial execution
   simpleFoam
   
   # Parallel execution
   mpirun -np 4 simpleFoam -parallel
   ```

3. **Reconstruct Results** [6]
   ```bash
   reconstructPar
   ```

## Post-Processing

### Using ParaView

1. **Create ParaView File** [13][14]
   ```bash
   touch case.foam
   ```

2. **Launch ParaView** [13]
   ```bash
   paraview case.foam
   ```

3. **Visualization Options** [13]:
   - Contour plots for pollutant concentration
   - Vector plots for velocity fields
   - Streamlines for flow visualization
   - Slice plots through building regions

### Command Line Post-Processing

1. **Sample Data** [13]
   ```bash
   postProcess -func probes
   postProcess -func surfaces
   ```

2. **Extract Data** [13]
   ```bash
   foamToVTK
   ```

## Troubleshooting

### Common Issues and Solutions

1. **Segmentation Fault** [15]
   - **Cause**: Memory access outside bounds
   - **Solution**: Recompile with debug mode and check array bounds
   ```bash
   export WM_COMPILE_OPTION=Debug
   ```

2. **Syntax Errors** [16]
   - **Cause**: Missing semicolons or brackets in dictionary files
   - **Solution**: Check error messages carefully and verify file syntax
   ```bash
   # Error usually indicates file and line number
   ```

3. **Mesh Quality Issues** [15]
   - **Cause**: Poor mesh quality affecting convergence
   - **Solution**: Refine mesh settings in `snappyHexMeshDict`
   ```bash
   checkMesh
   ```

4. **Convergence Problems** [15]
   - **Cause**: Inappropriate solver settings or boundary conditions
   - **Solution**: Adjust relaxation factors and solver tolerances in `fvSolution`

5. **WSL Installation Errors** [6]
   - **Error**: `WslRegisterDistribution failed with error: 0x80370114`
   - **Solution**: Enable "Windows Hypervisor Platform" in Windows features

### Debugging Tips

1. **Check Log Files** [15]
   ```bash
   tail -f log.simpleFoam
   ```

2. **Validate Case Setup** [17]
   ```bash
   foamToVTK -check
   ```

3. **Monitor Residuals** [15]
   ```bash
   foamLog log.simpleFoam
   gnuplot residuals.plt
   ```

## Results and Visualization

The simulation provides insights into [1][18]:
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

- **Mesh Size**: Balance between accuracy and computational cost [18]
- **Parallel Processing**: Use domain decomposition for large cases [18]
- **Time Step**: Ensure Courant number < 1 for stability [15]
- **Turbulence Modeling**: k-ε model recommended for urban flows [11]

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details [19][20].

## Acknowledgments

- OpenFOAM Foundation for the open-source CFD software [19]
- Contributors to the OpenFOAM community tutorials [21]
- Research works on urban air pollution modeling [1][2][3]

## Contact

For questions or issues, please open an issue on GitHub or contact the maintainer.

---

**Last Updated**: June 2025
