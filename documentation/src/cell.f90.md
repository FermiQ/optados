# `cell.f90` - Cell Information Module (`od_cell`)

## Overview

The `od_cell` module, defined in `cell.f90`, is central to managing geometric and reciprocal space information for the OptaDOS calculation. It handles the reading, processing, and storage of data related to the crystal cell, including lattice parameters, atomic positions, k-point grids, and symmetry operations.

## Key Components

The module `od_cell` defines several public variables to store cell-related data and subroutines to manipulate and report this data.

### Public Module Variables:

**Real Space Variables:**
*   `real_lattice(3,3)` (real(dp)): Stores the real space lattice vectors (e.g., rows or columns, units typically Angstrom after processing).
*   `recip_lattice(3,3)` (real(dp)): Stores the reciprocal space lattice vectors (units typically Angstrom<sup>-1</sup>).
*   `cell_volume` (real(dp)): The volume of the unit cell (units typically Angstrom<sup>3</sup>).

**Reciprocal Space Variables:**
*   `kpoint_r(:,:)` (real(dp), allocatable): Fractional coordinates of k-points.
*   `kpoint_r_cart(:,:)` (real(dp), allocatable): Cartesian coordinates of k-points.
*   `kpoint_weight(:)` (real(dp), allocatable): Weight for each k-point.
*   `num_kpoints_on_node(:)` (integer, allocatable): Number of k-points assigned to each MPI node.
*   `nkpoints` (integer): Total number of k-points.
*   `kpoint_grid_dim(3)` (integer): Dimensions of the Monkhorst-Pack k-point grid.

**Symmetry Operations:**
*   `num_crystal_symmetry_operations` (integer): Number of crystal symmetry operations.
*   `crystal_symmetry_disps(:,:)` (real(dp), allocatable): Displacement vectors for symmetry operations.
*   `crystal_symmetry_operations(:,:,:)` (real(dp), allocatable): Rotation/matrix part of symmetry operations.

**Atom Sites:**
*   `atoms_pos_frac(:,:,:)` (real(dp), allocatable): Fractional coordinates of atoms, dimensioned (3, max_sites_per_species, num_species).
*   `atoms_pos_cart(:,:,:)` (real(dp), allocatable): Cartesian coordinates of atoms.
*   `atoms_species_num(:)` (integer, allocatable): Number of atoms for each species.
*   `atoms_label(:)` (character(len=maxlen), allocatable): Label for each atomic species (e.g., "Si1").
*   `atoms_symbol(:)` (character(len=2), allocatable): Chemical symbol for each atomic species (e.g., "Si").
*   `num_atoms` (integer): Total number of atoms in the cell.
*   `num_species` (integer): Number of distinct atomic species.

### Public Subroutines:

*   **`subroutine cell_find_MP_grid(kpoints, num_kpts, kpoint_grid_dim, kpoint_offset)`**:
    *   Determines the Monkhorst-Pack (MP) grid dimensions (`kpoint_grid_dim`) and optionally the k-point offset (`kpoint_offset`) from a list of k-points.
    *   It considers time-reversal symmetry and attempts to deduce the underlying MP grid even with shifts.
    *   Handles special cases for small numbers of unique k-points.

*   **`subroutine cell_get_symmetry`**:
    *   Reads crystal symmetry operations from a `.sym` file (e.g., `seedname.sym`).
    *   Populates `num_crystal_symmetry_operations`, `crystal_symmetry_operations`, and `crystal_symmetry_disps`.
    *   If the `.sym` file is not found, it proceeds without symmetry information.
    *   Handles MPI broadcasting of symmetry data from the root node.

*   **`subroutine cell_read_cell`**:
    *   Reads atomic positions and symmetry operations from the CASTEP output cell file (`seedname-out.cell`).
    *   Parses `%block positions_frac/abs` and `%block symmetry_ops`.
    *   Populates atom-related arrays (`atoms_pos_frac_tmp`, `atoms_pos_cart_tmp`, `atoms_label_tmp`, etc., which are then processed by `cell_get_atoms` or similar logic) and symmetry arrays.
    *   *Note*: The provided code shows `cell_read_cell` primarily focused on reading the `-out.cell` file for atomic positions and symmetry, while `cell_get_atoms` reads the `.cell` file. There might be an overlap or specific division of responsibilities between these related to input file formats. This documentation reflects the content of `cell_read_cell` as presented.

*   **`subroutine cell_get_atoms`**:
    *   Reads atomic structure information from the input cell file (e.g., `seedname.cell`).
    *   Parses `%block positions_frac` or `%block positions_abs` to get atomic species labels and their coordinates.
    *   Handles unit conversions (e.g., Bohr to Angstrom if specified).
    *   Determines `num_atoms`, `num_species`.
    *   Populates `atoms_label`, `atoms_symbol`, `atoms_species_num`, `atoms_pos_frac`, and `atoms_pos_cart`.
    *   Sorts atoms by species.

*   **`subroutine cell_calc_lattice`**:
    *   Calculates the reciprocal lattice vectors (`recip_lattice`) and cell volume (`cell_volume`) from the `real_lattice` vectors.
    *   Converts `real_lattice` from Bohr to Angstrom (as it expects input in Bohr).
    *   Ensures `cell_volume` is positive.
    *   Scales `recip_lattice` by `2*pi/cell_volume`.

*   **`subroutine cell_report_parameters`**:
    *   Prints the real and reciprocal lattice vectors and the cell volume to the standard output unit.
    *   Formats the output for readability.

*   **`subroutine cell_dist`**:
    *   Distributes cell-related information (lattice vectors, volume, k-point info, atomic data, symmetry operations) from the root MPI node to all other nodes using `comms_bcast`.
    *   Allocates necessary arrays on non-root nodes before broadcasting.

## Important Variables/Constants

(See Public Module Variables section above)
*   `dp` (from `od_constants`): Double precision kind parameter.
*   `maxlen` (from `od_io`): Maximum length for character strings.
*   `bohr2ang` (from `od_constants`): Conversion factor from Bohr to Angstrom.

## Usage Examples

Typically, other modules within OptaDOS would `use od_cell` to access cell parameters and atomic information.

```fortran
program MyCalculation
  use od_cell
  use od_io, only : seedname, stdout
  use od_parameters, only : param_read ! To set up units etc.
  implicit none

  ! Minimal setup
  seedname = "mycrystal" ! Needs corresponding .cell file
  call param_read ! Assuming this sets up necessary units in od_parameters
                 ! and that real_lattice is populated (e.g. from .param or .cell file)

  ! If real_lattice is populated (e.g. after param_read or direct setting from a .cell file)
  ! call cell_calc_lattice ! Calculates recip_lattice and cell_volume

  ! call cell_get_atoms ! Reads atomic positions from mycrystal.cell

  ! if (on_root) then ! Assuming od_comms is also used
  !   call cell_report_parameters
  !   write(stdout,*) "Number of atoms: ", num_atoms
  !   write(stdout,*) "Number of species: ", num_species
  ! endif
  ! ... rest of the calculation using cell data ...
end program MyCalculation
```

## Dependencies and Interactions

*   **`od_constants`**: For `dp` (double precision kind) and `bohr2ang` conversion factor, `pi`.
*   **`od_io`**: For `maxlen`, file operations (`io_file_unit`, `io_error`), `seedname`, and `stdout`.
*   **`od_algorithms`**: Used by `cell_get_atoms` for coordinate conversions (`utility_cart_to_frac`, `utility_frac_to_cart`) and string manipulation (`utility_lowercase`).
*   **`od_comms`**: Used by `cell_get_symmetry` and `cell_dist` for MPI communication (`on_root`, `comms_bcast`).
*   **Input Files**:
    *   `seedname.cell`: Read by `cell_get_atoms` for atomic positions and lattice parameters (if not already set).
    *   `seedname-out.cell`: Read by `cell_read_cell` for atomic positions and symmetry operations (often a CASTEP output).
    *   `seedname.sym`: Optionally read by `cell_get_symmetry` for symmetry operations.

The module is fundamental for defining the geometric context of the electronic structure calculations performed by OptaDOS.
