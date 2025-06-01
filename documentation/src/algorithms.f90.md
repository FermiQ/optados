# `algorithms.f90` - Low-Level Algorithms Module (`od_algorithms`)

## Overview

The `od_algorithms` module, defined in `algorithms.f90`, provides a collection of low-level algorithms and utility functions that are not specific to electronic structure calculations but are used in various parts of the OptaDOS code. These include mathematical functions, sorting routines, string manipulations, and coordinate conversions.

## Key Components

The module `od_algorithms` makes the following routines publicly available:

*   **`function channel_to_am(no)`**:
    *   Converts an integer `no` representing an angular momentum channel (1-5) to its corresponding character ('s', 'p', 'd', 'f', 'g').
    *   Parameters:
        *   `no` (integer, intent(in)): The angular momentum channel number.
    *   Returns: `character(len=1)`: The single character representation.

*   **`function gaussian(m, w, x)`**:
    *   Calculates the value of a Gaussian function at point `x` with a given mean `m` and width `w`.
    *   It handles cases where the exponent is very small by returning 0.0 to avoid underflow/overflow.
    *   Parameters:
        *   `m` (real(dp), intent(in)): Mean of the Gaussian.
        *   `w` (real(dp), intent(in)): Width (standard deviation) of the Gaussian.
        *   `x` (real(dp), intent(in)): Position at which to evaluate the Gaussian.
    *   Returns: `real(dp)`: The value of the Gaussian function.

*   **`subroutine heap_sort(num_items, weight)`**:
    *   Sorts an array `weight` of real numbers into descending order using the heap sort algorithm.
    *   Parameters:
        *   `num_items` (integer, intent(in)): The number of items in the `weight` array.
        *   `weight` (real(dp), dimension(num_items), intent(inout)): The array of weights to be sorted.

*   **`function utility_lowercase(string)`**:
    *   Converts an input character string to lowercase.
    *   Parameters:
        *   `string` (character(len=*), intent(in)): The input string.
    *   Returns: `character(len=maxlen)`: The lowercase version of the string (padded to `maxlen` from `od_io`).

*   **`subroutine utility_cart_to_frac(frac, cart, real_lat)`**:
    *   Converts a 3D vector from Cartesian coordinates `cart` to fractional coordinates `frac` using the real space lattice vectors `real_lat`.
    *   Parameters:
        *   `real_lat` (real(dp), intent(in), dimension(3,3)): Real space lattice vectors (columns or rows depending on convention).
        *   `frac` (real(dp), intent(out), dimension(3)): The resulting fractional coordinates.
        *   `cart` (real(dp), intent(in), dimension(3)): The input Cartesian coordinates.
    *   *Correction*: The original code comment says "Convert from fractional to Cartesian", but the argument names and calculation `cart(i) = real_lat(1,i)*frac(1) + ...` imply it converts fractional `frac` to Cartesian `cart`. The documentation reflects the likely implementation based on standard conventions. If `real_lat` stores lattice vectors as columns, `cart = real_lat * frac`. If rows, `cart = frac^T * real_lat`. The code seems to use `cart(i) = sum_j real_lat(j,i) * frac(j)`.

*   **`subroutine utility_frac_to_cart(frac, cart, real_lat)`**:
    *   Converts a 3D vector from fractional coordinates `frac` to Cartesian coordinates `cart` using the real space lattice vectors `real_lat`.
    *   Parameters:
        *   `real_lat` (real(dp), intent(in), dimension(3,3)): Real space lattice vectors.
        *   `frac` (real(dp), intent(in), dimension(3)): The input fractional coordinates.
        *   `cart` (real(dp), intent(out), dimension(3)): The resulting Cartesian coordinates.
    *   *Note*: This appears to be the correct fractional to Cartesian. The previous entry might be Cartesian to Fractional or there's a naming confusion in the original source's comments/arguments. This documentation describes the function signature as `utility_frac_to_cart(frac, cart, real_lat)`.

*   **`subroutine utility_cart_to_frac(cart, frac, recip_lat)`**:
    *   Converts a 3D vector from Cartesian coordinates `cart` to fractional coordinates `frac` using the reciprocal space lattice vectors `recip_lat`. The result is scaled by `1/(2*pi)`.
    *   Parameters:
        *   `recip_lat` (real(dp), intent(in), dimension(3,3)): Reciprocal space lattice vectors.
        *   `cart` (real(dp), intent(in), dimension(3)): The input Cartesian coordinates.
        *   `frac` (real(dp), intent(out), dimension(3)): The resulting fractional coordinates.

*   **`function algorithms_erf(x)`**:
    *   Calculates the error function `erf(x)`.
    *   Uses different polynomial approximations for different ranges of `abs(x)` for accuracy and efficiency. Based on NSWC Mathematics Subroutine Library.
    *   Parameters:
        *   `x` (real(dp), intent(in)): The input value.
    *   Returns: `real(dp)`: The value of `erf(x)`.

*   **`subroutine algor_dist_array(num_elements, elements_per_node)`**:
    *   Distributes a total `num_elements` across `num_nodes` (obtained from `od_comms`).
    *   Calculates how many elements each node should handle and stores this in the `elements_per_node` array.
    *   Handles cases where `num_elements < num_nodes` by issuing an error.
    *   Distributes any remainder elements one by one to the initial nodes.
    *   Parameters:
        *   `num_elements` (integer, intent(in)): Total number of elements to distribute.
        *   `elements_per_node` (integer, allocatable, intent(out), dimension(:)): Array (sized 0 to num_nodes-1) containing the number of elements for each node.

## Important Variables/Constants

*   The module itself does not define public constants, but it uses `dp` (double precision kind) and `inv_sqrt_two_pi` from the `od_constants` module.
*   It also uses `maxlen` from `od_io` for string operations.
*   The `algorithms_erf` function contains internal constants (arrays A, B, P, Q, R, S, C) for its polynomial approximations.

## Usage Examples

**Gaussian function:**
```fortran
use od_algorithms, only : gaussian
use od_constants, only : dp
real(kind=dp) :: result, mean_val, width_val, x_val
mean_val = 0.0_dp
width_val = 1.0_dp
x_val = 0.5_dp
result = gaussian(mean_val, width_val, x_val)
! result will be the value of Gaussian(0,1) at x=0.5
```

**Heap Sort:**
```fortran
use od_algorithms, only : heap_sort
use od_constants, only : dp
integer, parameter :: n = 5
real(kind=dp) :: my_array(n) = [5.0_dp, 1.0_dp, 4.0_dp, 2.0_dp, 3.0_dp]
call heap_sort(n, my_array)
! my_array will now be [5.0, 4.0, 3.0, 2.0, 1.0]
```

**Lowercase:**
```fortran
use od_algorithms, only : utility_lowercase
character(len=20) :: original_string, lower_string
original_string = "TestSTRING"
lower_string = utility_lowercase(original_string)
! lower_string will be "teststring" (possibly trimmed/adjusted)
```

## Dependencies and Interactions

*   **`od_constants`**: Uses `dp` for precision and `inv_sqrt_two_pi` for the Gaussian function, and `twopi` for coordinate conversion.
*   **`od_io`**: Uses `maxlen` for string length in `utility_lowercase` and `io_error` for error reporting in `algor_dist_array`.
*   **`od_comms`**: Uses `num_nodes` in `algor_dist_array` to determine the number of MPI nodes for distributing array elements.

The functions and subroutines in this module are generally self-contained or rely on basic constants and types from other utility modules.
