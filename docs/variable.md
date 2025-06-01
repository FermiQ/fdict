# `variable.fypp`

## Overview

The `variable.fypp` file defines a powerful generic variable type (`variable_t`) in Fortran. This type is designed to hold data of almost any kind, including intrinsic Fortran types (integers, reals, complex, logicals, characters) of various precisions and ranks, as well as user-defined types (though support for user-defined types requires external routines).

The `variable_t` acts as a container. It stores a pointer to the actual data and uses a type-transfer mechanism by encoding the data's type and pointer information into a character array (`enc`). This allows for dynamic typing capabilities within a statically-typed language like Fortran. This module is fundamental to `dictionary.fypp`, enabling its dictionaries to store heterogeneous data types.

## Key Components

### `variable_t`
A derived type that serves as the generic data container.
  - `t`: A character string (length `VARIABLE_TYPE_LENGTH`) that stores a short identifier for the data type being held (e.g., "r40" for a scalar single-precision real, "i81" for a 1D array of 64-bit integers).
  - `enc`: An allocatable character array (`character(len=1), dimension(:)`) that stores the encoded representation of the pointer to the actual data.

### Key Operations and Interfaces

- **`which`**: Returns the type string (e.g., "r40", "i81") of the data stored in a `variable_t` or of a given Fortran variable.
- **`delete`**: Deallocates the data pointed to by `variable_t` (if it's responsible for it) and resets the variable.
- **`nullify`**: Resets the `variable_t` to an empty state, deallocating the `enc` array but not necessarily the data it points to (similar to Fortran's `nullify` for pointers).
- **`empty`**: Checks if the `variable_t` is empty (i.e., not allocated or nullified).
- **`print`**: Prints the type string of the `variable_t` to standard output.
- **`associate_type`**: Allows associating a raw encoded character array (`enc`) with a `variable_t`, typically used for storing user-defined types where the encoding is handled externally.
- **`enc`**: Retrieves the raw character encoding from a `variable_t`.
- **`size_enc`**: Returns the size of the `enc` array.
- **`cpack`**: Converts a `character(len=*)` string to a `character(len=1), dimension(:)` array.
- **`cunpack`**: Converts a `character(len=1), dimension(:)` array back to a `character(len=*)` string.
- **`assign`**:
    - `assign(variable_out, variable_in)`: Copies the content from one `variable_t` to another (deep copy).
    - `assign(variable_out, fortran_variable_in)`: Stores a copy of a Fortran variable into a `variable_t`.
    - `assign(fortran_variable_out, variable_in)`: Retrieves a copy of the data from a `variable_t` into a Fortran variable.
- **`associate`**:
    - `associate(variable_out, variable_in)`: Makes one `variable_t` point to the same data as another `variable_t` (shallow copy of the descriptor).
    - `associate(variable_out, fortran_target_in)`: Makes a `variable_t` point to a Fortran variable (which must have the `TARGET` attribute).
    - `associate(fortran_pointer_out, variable_in)`: Makes a Fortran pointer point to the data held by a `variable_t`.
- **`associatd`**: Checks if a `variable_t` is associated with a given Fortran pointer or if two `variable_t` instances are associated with the same data.

## Important Variables/Constants

### `VARIABLE_TYPE_LENGTH`
An integer parameter defining the maximum length of the type specifier string (e.g., "r82", "c40") stored in `variable_t%t`. The value is automatically determined based on the maximum length of generated type names.

### `VARIABLE_ENC_TYPE`
A `character(len=1), dimension(1)` constant, likely used internally for `transfer` operations related to pointer encoding.

### Preprocessor Macros for Type/Rank Combinations
The file extensively uses `fypp` preprocessor loops (e.g., `#:for sh, k, t, minr, maxr in ALL_SHORTS_KINDS_TYPES_RANKS`) to generate specific `assign`, `associate`, `which`, and `associatd` procedures for a multitude of type, kind, and rank combinations. `ALL_SHORTS_KINDS_TYPES_RANKS` is defined in `fdict_common.fypp`.

## Usage Examples

```fortran
program test_variable
  use variable
  use fdict_types
  implicit none

  type(variable_t) :: v1, v2
  integer(kind=int32) :: i_scalar, i_array_1d(3)
  real(kind=real64), target :: r_target_scalar
  real(kind=real64), pointer :: r_ptr_scalar

  ! Assign a scalar integer to v1
  i_scalar = 123
  call assign(v1, i_scalar)
  write(*,*) 'v1 type:', which(v1) ! Output: v1 type: i40 (or similar for int32 scalar)

  ! Retrieve the integer from v1
  call assign(i_scalar, v1)
  write(*,*) 'Retrieved i_scalar:', i_scalar ! Output: Retrieved i_scalar: 123

  ! Assign a real array to v1
  i_array_1d = [10, 20, 30]
  call assign(v1, i_array_1d)
  write(*,*) 'v1 type (array):', which(v1) ! Output: v1 type: i41 (or similar for int32 1D array)

  ! Associate v1 with a real target
  r_target_scalar = 3.14159_real64
  call associate(v1, r_target_scalar)
  write(*,*) 'v1 type (associated real):', which(v1) ! Output: v1 type: r80 (or similar)

  ! Modify the target, v1 should reflect the change
  r_target_scalar = 2.71828_real64
  call associate(r_ptr_scalar, v1)
  if (associated(r_ptr_scalar)) then
    write(*,*) 'Associated r_ptr_scalar:', r_ptr_scalar ! Output: Associated r_ptr_scalar: 2.71828
  end if

  ! Assign v1 (pointing to r_target_scalar) to v2 (deep copy)
  call assign(v2, v1)
  ! Now modify r_target_scalar again. v2 should hold the value from before this change.
  r_target_scalar = 1.61803_real64
  call associate(r_ptr_scalar, v2)
  if (associated(r_ptr_scalar)) then
     ! This will print 2.71828 because v2 made a copy
    write(*,*) 'Copied value in v2 (r_ptr_scalar):', r_ptr_scalar
  end if
  call associate(r_ptr_scalar, v1)
  if (associated(r_ptr_scalar)) then
    ! This will print 1.61803 because v1 is still associated
    write(*,*) 'Original value in v1 (r_ptr_scalar):', r_ptr_scalar
  end if


  ! Clean up
  call delete(v1)
  call delete(v2)
  write(*,*) 'v1 empty after delete?', empty(v1) ! Output: v1 empty after delete? T

end program test_variable
```

## Dependencies and Interactions

- **`fdict_types`**: Uses type kind definitions (e.g., `int32`, `real64`) from `fdict_types.fypp` for declaring internal pointers and for generating type-specific procedures.
- **`fdict_common.fypp`**: Includes `fdict_common.fypp`, which provides common preprocessor definitions, macros (like `_pure`, `_elemental`), and importantly, the `ALL_SHORTS_KINDS_TYPES_RANKS` list that drives the generation of numerous type-specific procedures.
- **Fortran Intrinsic `transfer`**: Relies heavily on the `transfer` intrinsic to convert pointer addresses and type information into the `enc` character array and back. This is a core mechanism for its type-erasure capability.
- **Memory Management**:
    - When `assign` is used to put data into `variable_t`, `variable_t` typically allocates its own memory and copies the data.
    - When `associate` is used with a Fortran `TARGET` variable, `variable_t` stores a pointer to the existing variable. The user must ensure the `TARGET` variable remains allocated and valid as long as `variable_t` points to it.
    - `delete` should be called on a `variable_t` to deallocate the data it owns.
    - `nullify` makes the `variable_t` empty but doesn't deallocate the data if it was merely associated with an external target.
- **`dictionary.fypp`**: The `dictionary_t` type in `dictionary.fypp` uses `variable_t` to store its values, allowing the dictionary to hold items of different data types.
