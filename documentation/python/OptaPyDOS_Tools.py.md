# `OptaPyDOS_Tools.py` - Python Utilities for OptaDOS

## Overview

`OptaPyDOS_Tools.py` provides a set of Python functions designed to simplify common tasks when working with the OptaDOS library through its Python interface (`OptaPyDOS`). These tools offer a more convenient way to perform setup operations and manage calculation parameters, abstracting some of the direct calls to the underlying wrapped Fortran routines.

## Key Components

*   **`od_OptaPyDOS_tools_version` (float)**:
    *   A module-level variable indicating the version of these tools (e.g., `1.0`).

*   **`def od_setup_from_file(seedname)`**:
    *   **Purpose**: Initializes an OptaDOS calculation environment based on a given `seedname`. This function mimics the initial setup steps performed by the main `optados` Fortran program.
    *   **Actions**:
        1.  Imports `OptaPyDOS` as `opd`.
        2.  Sets up standard output (`opd.od_io.stdout`) and error (`opd.od_io.stderr`) file units.
        3.  Initializes MPI communications via `opd.od_comms.comms_setup()`.
        4.  Sets the global `seedname` in `opd.od_io`.
        5.  Reads calculation parameters using `opd.od_parameters.param_read()`.
        6.  Writes parameter headers and values to the output via `opd.od_parameters.param_write_header()` and `opd.od_parameters.param_write()`.
        7.  Reads band energy data using `opd.od_electronic.elec_read_band_energy()`.
        8.  Calculates lattice parameters with `opd.od_cell.cell_calc_lattice()`.
        9.  Reports cell and electronic parameters using `opd.od_cell.cell_report_parameters()` and `opd.od_electronic.elec_report_parameters()`.
        10. Distributes parameters and cell information to other MPI nodes (if applicable) using `opd.od_parameters.param_dist()` and `opd.od_cell.cell_dist()`.
    *   **Parameters**:
        *   `seedname` (str): The base name for input files (e.g., `.odi`, `.cell`, `.param` files).

*   **`def od_set_efermi()`**:
    *   **Purpose**: Manages the Fermi energy for calculations. It calls internal OptaDOS routines to determine the Fermi energy and optionally shifts the energy scale if requested by user parameters.
    *   **Actions**:
        1.  Imports `OptaPyDOS` as `opd`.
        2.  Calls `opd.od_dos_utils.dos_utils_set_efermi()` to determine the appropriate Fermi energy based on input parameters and electronic structure data.
        3.  Retrieves the determined chemical potential (Fermi energy) from `opd.od_electronic.efermi`.
        4.  If the user parameter `opd.od_parameters.set_efermi_zero` is true, it shifts the energy scale array `opd.od_dos_utils.e` by subtracting the chemical potential. The chemical potential itself is then set to 0 for consistency in subsequent references within this shifted context.
    *   **Returns**:
        *   `chemical_potential` (float): The value of the Fermi energy used (0.0 if `set_efermi_zero` was true, otherwise the determined Fermi energy).

## Important Variables/Constants

*   **`od_OptaPyDOS_tools_version`**: As described above.

## Usage Examples

**Setting up a calculation:**

```python
import OptaPyDOS_Tools as odtools

# Define the seedname for your CASTEP/OptaDOS input files
my_seedname = "Si_bulk"

# Perform initial setup (reads .odi, .param, .cell, .bands etc.)
odtools.od_setup_from_file(my_seedname)

# At this point, parameters are read, and basic data is loaded and distributed.
# Further calls to OptaPyDOS modules can be made to perform calculations.
```

**Setting/Adjusting the Fermi energy:**

```python
import OptaPyDOS_Tools as odtools
import OptaPyDOS as opd # Assuming setup has been done

# Ensure parameters are loaded, including set_efermi_zero if needed
# For example, to set Fermi energy to zero in output spectra:
# opd.od_parameters.set_efermi_zero = True

effective_efermi = odtools.od_set_efermi()
print(f"Effective Fermi energy for spectra: {effective_efermi} eV")

# Now, opd.od_dos_utils.e (the energy scale for DOS) will be shifted if set_efermi_zero was True.
```

## Dependencies and Interactions

*   **`OptaPyDOS` (imported as `opd`)**: This is the primary dependency. `OptaPyDOS_Tools.py` makes extensive calls to various modules and functions within `OptaPyDOS` (e.g., `opd.od_io`, `opd.od_comms`, `opd.od_parameters`, `opd.od_electronic`, `opd.od_cell`, `opd.od_dos_utils`).
*   **OptaDOS Fortran Library**: Indirectly, it depends on the compiled OptaDOS Fortran library, as `OptaPyDOS` is a wrapper around it.
*   **Input Files**: `od_setup_from_file` relies on the presence of OptaDOS input files (`<seedname>.odi`, `<seedname>.param`, etc.) and associated electronic structure files (e.g., `<seedname>.bands` from a CASTEP calculation).

These tools act as a higher-level scripting interface, making it easier to drive OptaDOS calculations from Python by bundling sequences of calls to the `f90wrap`-generated `OptaPyDOS` module.
