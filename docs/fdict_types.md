# `fdict_types.fypp`

## Overview

The `fdict_types.fypp` file serves as a centralized module for defining various Fortran data type kinds. Its primary role is to ensure consistency in data representations, particularly for numerical precision (integers, reals) and logical types, across the fdict project. It leverages Fortran's intrinsic `iso_fortran_env` module where available and provides fallbacks using `selected_real_kind` and `selected_int_kind` for environments where `iso_fortran_env` might not be fully supported or for specific precision requirements not directly covered by it (e.g., `real80`).

This module is crucial for writing portable and reliable Fortran code that behaves predictably across different compilers and hardware architectures by providing named constants for kind parameters.

## Key Components

This module primarily defines parameters for various data type kinds. These parameters are then used in declarations of variables throughout the fdict library.

### Integer Kinds
- `int8`: 8-bit integer (if `FDICT_WITH_INT8` is enabled).
- `int16`: 16-bit integer (if `FDICT_WITH_INT16` is enabled).
- `int32`: 32-bit integer.
- `int64`: 64-bit integer.

### Real Kinds
- `real32`: Single-precision floating-point (approximately 6-7 decimal digits).
- `real64`: Double-precision floating-point (approximately 15-17 decimal digits).
- `real80`: Extended-precision floating-point (typically 80-bit, providing higher precision than `real64`, if `FDICT_WITH_REAL80` is enabled). Often corresponds to `selected_real_kind(18)`.
- `real128`: Quadruple-precision floating-point (if `FDICT_WITH_REAL128` is enabled). Often corresponds to `selected_real_kind(30)`.

### Logical Kinds
- `log8`: 8-bit logical (if `FDICT_WITH_LOG8` is enabled). Typically maps to `int8`.
- `log16`: 16-bit logical (if `FDICT_WITH_LOG16` is enabled). Typically maps to `int16`.
- `log32`: 32-bit logical. Typically maps to `int32`.
- `log64`: 64-bit logical (if `FDICT_WITH_LOG64` is enabled). Typically maps to `int64`.

### ISO C Binding Kinds
- If `FDICT_WITH_ISO_C` is enabled, this module also makes ISO C binding kinds like `c_ptr`, `c_funptr` available through `use, intrinsic :: iso_c_binding`.

## Important Variables/Constants

The "Key Components" section lists the primary constants defined in this file. These are all `integer, parameter` constants representing kind type parameters. Their availability and specific values can depend on preprocessor flags (`FDICT_WITH_...`) and the Fortran compiler's capabilities.

- **`FDICT_WITH_ISO_ENV`**: Preprocessor flag. If true, types are sourced from `iso_fortran_env`. Otherwise, `selected_*_kind` is used.
- **`FDICT_WITH_INT8`, `FDICT_WITH_INT16`**: Control inclusion of `int8`, `int16`.
- **`FDICT_WITH_REAL80`, `FDICT_WITH_REAL128`**: Control inclusion of `real80`, `real128`.
- **`FDICT_WITH_LOG8`, `FDICT_WITH_LOG16`, `FDICT_WITH_LOG64`**: Control inclusion of `log8`, `log16`, `log64`.
- **`FDICT_WITH_ISO_C`**: Controls the use of `iso_c_binding` for C interoperability types.

## Usage Examples

```fortran
module my_module
  use fdict_types
  implicit none

  integer(kind=int32) :: my_standard_integer
  real(kind=real64) :: my_double_precision_real
  logical(kind=log8) :: my_small_logical

  ! Check if extended precision is available before using it
#:if FDICT_WITH_REAL80 == 1
  real(kind=real80) :: my_extended_real
#:else
  ! Fallback or error if real80 is not available
  real(kind=real64) :: my_extended_real ! Using real64 as fallback
  ! write(*,*) "Warning: real80 not available, using real64 instead."
#:endif

  my_standard_integer = 12345
  my_double_precision_real = 3.141592653589793_real64
  my_small_logical = .true.

  ! ... further code using these variables
end module my_module
```

## Dependencies and Interactions

- **`fdict_common.fypp`**: This file includes `fdict_common.fypp`. This inclusion is essential as `fdict_common.fypp` defines the preprocessor variables (e.g., `FDICT_WITH_ISO_ENV`, `FDICT_WITH_REAL80`) that control which type kinds are actually defined and how they are defined (either via `iso_fortran_env` or `selected_*_kind`).
- **`iso_fortran_env` (Intrinsic Module)**: Conditionally used if `FDICT_WITH_ISO_ENV` is true. Provides standard kind parameters like `int32`, `int64`, `real32`, `real64`.
- **`iso_c_binding` (Intrinsic Module)**: Conditionally used if `FDICT_WITH_ISO_C` is true. Provides types for C interoperability.
- **Fortran Compiler**: The actual bit sizes and availability of certain kinds (especially `real80`, `real128`) depend on the capabilities of the Fortran compiler being used. This module attempts to provide a consistent way to access them.
- **Other modules in fdict**: All other modules in the fdict library that declare variables with specific precision or type kinds (e.g., `variable.fypp`, `dictionary.fypp`) will `use fdict_types` to access these definitions. This ensures that the entire library uses a consistent set of type kinds.
