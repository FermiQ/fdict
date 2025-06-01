# `dictionary.fypp`

## Overview

The `dictionary.fypp` file implements a generic dictionary type (`dictionary_t`) in Fortran. This dictionary is similar in concept to Python dictionaries or hash maps in other languages. It allows storing key-value pairs where keys are strings and values can be of any data type supported by the `variable_t` type from `variable.fypp`. This module provides the core functionality for creating, manipulating, and accessing data within these dictionaries.

## Key Components

### `dictionary_t`
A derived type representing the dictionary. It internally uses a linked list of `dictionary_entry_t` to store key-value pairs.
  - `first`: A pointer to the first `dictionary_entry_t` in the linked list.
  - `len`: An integer storing the number of entries in the dictionary.

### `dictionary_entry_t`
A private derived type representing an entry in the dictionary.
  - `key`: A character string (up to `DICTIONARY_KEY_LENGTH`) storing the key.
  - `value`: A `variable_t` type storing the value associated with the key.
  - `hash`: An integer storing the hash value of the key for efficient lookups.
  - `next`: A pointer to the next `dictionary_entry_t` in the linked list.

### Key Operations (Operators and Procedures)

- **`LEN` (interface `len`)**: Returns the number of entries in the dictionary (using the internal counter).
- **`LLEN` (interface `llen`)**: Returns the number of entries by traversing the linked list (actual count).
- **`print`**: Prints all keys, their data types, and hash numbers to standard output.
- **`operator(//)`**: Concatenates two dictionaries or extends a dictionary with new key-value pairs.
- **`operator(.KEY.)`**: Returns the key of the current top entry in a dictionary (typically used in loops).
- **`operator(.IN.)`**: Checks if a key exists in the dictionary.
- **`operator(.NIN.)`**: Checks if a key does not exist in the dictionary.
- **`operator(.VAL.)`**: Returns the value associated with a key (by copy).
- **`operator(.VALP.)`**: Returns a pointer to the value associated with a key.
- **`operator(.HASH.)` / `hash`**: Returns the hash value of the dictionary's first item's key.
- **`operator(.EQ.)`**: Checks if two dictionaries have all the same keys (compares lengths and hash values of keys).
- **`operator(.NE.)`**: Checks if two dictionaries do not share any common keys.
- **`operator(.NEXT.)` / `next`**: Advances to the next entry in the dictionary (used for looping).
- **`operator(.FIRST.)` / `first`**: Returns the first entry of a dictionary.
- **`operator(.EMPTY.)` / `empty`**: Checks if the dictionary is empty or if a dictionary entry's value is empty.
- **`hash_coll`**: Calculates the number of hash collisions in the dictionary.
- **`delete`**: Deletes an entry by key or clears the entire dictionary.
- **`pop`**: Removes an entry by key and returns its value.
- **`copy`**: Creates a deep copy of a dictionary.
- **`nullify`**: Nullifies a dictionary (clears all entries and releases memory) or nullifies a specific key.
- **`extend`**: Extends a dictionary with entries from another dictionary (similar to `//` but as a subroutine).
- **`which`**: Returns the data type string of the value associated with a key, or the first entry if no key is provided.
- **`operator(.KV.)`**: Creates a dictionary entry by copying the value. Used like `'mykey'.KV.myvariable`.
- **`operator(.KVP.)`**: Creates a dictionary entry by associating (pointing to) the value. Used like `'mykey'.KVP.my_target_variable`.
- **`assign`**: Assigns a value from the dictionary to a variable (retrieves by copy).
- **`associate`**: Associates a variable with a value in the dictionary (retrieves by pointer).

## Important Variables/Constants

### `DICTIONARY_KEY_LENGTH`
An integer parameter defining the maximum character length for keys in the dictionary. Default is 48.

### `DICTIONARY_NOT_FOUND`
A character string parameter returned by some functions when a key is not found. Default is `'ERROR: key not found'`.

### `HASH_ALGO`
A preprocessor variable (defaulting to 0) that determines the hashing algorithm used.
  - `0`: FNV-1a 32-bit hash algorithm.
  - `-1`: A custom hash algorithm (notes indicate it has many collisions).

## Usage Examples

```fortran
program test_dictionary
  use dictionary
  use variable
  implicit none

  type(dictionary_t) :: dict1, dict2, dict3
  integer :: i_val
  real, target :: r_val_target
  real, pointer :: r_val_ptr

  ! Create dictionary entries
  dict1 = ('MeaningOfLife'.KV.42)
  r_val_target = 3.14159
  dict1 = dict1 // ('Pi'.KVP.r_val_target)

  ! Check length
  write(*,*) 'Length of dict1:', LEN(dict1) ! Output: Length of dict1: 2

  ! Check if a key exists
  if ('Pi' .IN. dict1) then
    write(*,*) 'Pi exists in dict1'
  end if

  ! Get a value by copy
  call assign(i_val, dict1, key='MeaningOfLife')
  write(*,*) 'MeaningOfLife =', i_val ! Output: MeaningOfLife = 42

  ! Get a value by pointer
  call associate(r_val_ptr, dict1, key='Pi')
  if (associated(r_val_ptr)) then
    write(*,*) 'Pi (from pointer) =', r_val_ptr ! Output: Pi (from pointer) = 3.14159
    r_val_target = 2.71828 ! Modify the original target
    write(*,*) 'Pi (after modifying target) =', r_val_ptr ! Output: Pi (after modifying target) = 2.71828
  end if

  ! Print dictionary contents
  call print(dict1)

  ! Delete an entry
  call delete(dict1, key='MeaningOfLife')
  write(*,*) 'Length of dict1 after delete:', LEN(dict1) ! Output: Length of dict1 after delete: 1

  ! Nullify the dictionary
  call nullify(dict1)
  write(*,*) 'Is dict1 empty after nullify?', .EMPTY. dict1 ! Output: Is dict1 empty after nullify? T

end program test_dictionary
```

## Dependencies and Interactions

- **`fdict_types`**: Uses type definitions from `fdict_types.fypp` (e.g., for precision of internal variables if any, though not explicitly shown for `dictionary_t` itself, it's a common dependency in the project).
- **`variable`**: Heavily relies on `variable.fypp` for the `variable_t` type, which is used to store the actual values in the dictionary. This allows the dictionary to be generic and hold various data types.
- **`fdict_common.fypp`**: This file includes `fdict_common.fypp`, which likely provides common preprocessor definitions and macros used within `dictionary.fypp`, such as `_pure`, `_elemental`, and `ALL_SHORTS_KINDS_TYPES_RANKS` for code generation.
- **Hashing Algorithm**: The choice of `HASH_ALGO` affects the performance and collision rate of dictionary lookups.
- **Memory Management**: Users must be careful with concatenated dictionaries. The documentation within the code suggests nullifying original dictionaries after concatenation to avoid issues (`d1 = d2 // d3; call nullify(d2); call nullify(d3)`). Also, when using `.KVP.` to store pointers, the pointed-to data must remain valid for the lifetime of its use in the dictionary.
