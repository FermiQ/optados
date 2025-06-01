# `optados.f90` - Main Program

## Overview

`optados.f90` is the main program file for the OptaDOS code. It orchestrates the overall workflow of the calculation, initializing parameters, reading input data, calling various computational modules, and finalizing the output. OptaDOS is designed to calculate optical and density of states (DOS) related properties from electronic structure calculations.

## Key Components

*   **`program optados`**: The main program block.
    *   Initializes MPI communications if applicable (`comms_setup`).
    *   Handles command-line arguments like `--help` and `--version` through `help_output` and `version_output` subroutines.
    *   Sets up error (`.opt_err`) and output (`.odo`) files using the provided seedname.
    *   Reads user parameters (`param_read`).
    *   Reads electronic band structure data (`elec_read_band_energy` or `elec_read_band_energy_ordered`).
    *   Calculates lattice parameters (`cell_calc_lattice`).
    *   Distributes parameters and cell information to MPI nodes (`param_dist`, `cell_dist`).
    *   Conditionally calls calculation routines based on user parameters:
        *   `pdos_calculate` (Projected Density of States)
        *   `pdis_calculate` (Projected Dispersion)
        *   `core_calculate` (Core Level Spectra)
        *   `dos_calculate` (Density of States)
        *   `optics_calculate` (Optical Properties)
        *   `jdos_calculate` (Joint Density of States)
    *   Deallocates parameters (`param_dealloc`).
    *   Finalizes MPI communications (`comms_end`).
*   **`subroutine help_output`**:
    *   Prints usage information and exits.
    *   Uses `od_constants` for version and copyright information.
*   **`subroutine version_output`**:
    *   Prints detailed version and build information and exits.
    *   Uses `od_build` and `od_constants`.

## Important Variables/Constants

*   **`seedname` (character)**: The base name for input/output files, derived from command-line arguments.
*   **`stdout`, `stderr` (integer)**: File unit numbers for standard output and standard error, respectively.
*   **`time0`, `time1` (real(dp))**: Variables used for timing sections of the code.
*   **`odo_found` (logical)**: Flag indicating if an existing `.odo` output file was found.
*   **`pdos`, `pdis`, `dos`, `jdos`, `core`, `optics` (logical)**: Control flags read from parameters, determining which calculation tasks to perform. These are typically part of the `od_parameters` module but control the main program flow.
*   **`iprint` (integer)**: Verbosity level for printing output, from `od_parameters`.
*   **`on_root` (logical)**: From `od_comms`, true if the current MPI process is the root process.
*   **`num_nodes` (integer)**: From `od_comms`, the total number of MPI nodes.

## Usage Examples

OptaDOS is typically run from the command line:

```bash
optados <seedname>
```

Where `<seedname>` is the prefix for the input files (e.g., `Si_castep` if input files are `Si_castep.param`, `Si_castep.cell`, etc.).

The program can also be run with:

```bash
optados --help  # To display help information
optados --version # To display version information
```

## Dependencies and Interactions

`optados.f90` is the central coordinating program and depends on almost all other `od_*` modules:

*   **`od_comms`**: For MPI communication setup, control, and finalization.
*   **`od_constants`**: For physical constants and version information.
*   **`od_io`**: For file I/O operations (getting seedname, file units, date/time).
*   **`od_parameters`**: For reading, writing, and distributing user-defined parameters. This module provides the flags (e.g., `dos`, `optics`) that control the execution path.
*   **`od_cell`**: For cell-related calculations and reporting.
*   **`od_electronic`**: For reading band energies and other electronic structure data.
*   **`od_dos`**: For Density of States calculations.
*   **`od_jdos`**: For Joint Density of States calculations.
*   **`od_core`**: For Core Level Spectra calculations.
*   **`od_pdos`**: For Projected Density of States calculations.
*   **`od_pdis`**: For Projected Dispersion calculations.
*   **`od_optics`**: For Optical Properties calculations.
*   **`od_build`**: For accessing build-specific information (compiler, date, etc.).

The program flow is sequential, with initialization steps, followed by a series of conditional calls to calculation modules based on input parameters, and finally a cleanup/finalization phase.
