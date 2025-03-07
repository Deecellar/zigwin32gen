# zigwin32gen

Autogenerated Zig bindings for Win32 using https://github.com/marlersoft/win32json

## How to generate the Windows Zig bindings

Download zig, currently tested on version: 0.11.0-dev.2619+bd3e248c7

First, clone the zigwin32 repo from the root of this repository to provide a location for the generated files:

```
git clone https://github.com/marlersoft/zigwin32
```

Then run the following command to generate the latest bindings:

```
zig build
```

On the first run, running this command will provide an error message saying that the `win32json` dependency is missing, and will provide you with a git command to clone it and checkout the expected version.

Once the build task completes, you can view your generated code in the `zigwin32` subrepository that you cloned earlier.

## win32json vs C/C++ Win32 SDK

The win32metadata project from which win32json is generated does not expose the exact same interface as the classic C/C++ header files did.  One thing I'd like is to support is to be able to take a Windows C/C++ example and port it to Zig without changing any names/types or the way it interfaces with the Windows API.  To support this I'll need to:

* Support a set of modules with the same structure as the Window headers.  For example, importing the `windows` module should have all the same declarations as the `windows.h` header.
* Generate structs with the same field names (hungarian notation)
* Support code that can be compiled for either Ascii or Wide characters.  Provide mappings like `CreateFile` to `CreateFileA` or `CreateFileW` based on this configuration.


For example, intead of `DWORD` or `UINT32`, I would generate `u32`.  Instead of field names with hungarian notation, I would generate them with snake case.  These would only be cosmetic changes, where it would look more natural to use the Windows API directly alongside other Zig APIs.  The bigger advantage to this is not having to track all the different ways to declare the same type, for example, a developer having to know that `DWORD`, `UINT32`, `ULONG`, `UINT` are all the same as `u32`.
* I may want to be able support different "views" into the api that change things like how to import the api.  These extra views should not add to compile-time unless they are being used.
* I may want to consider also generating wrapper APIs?  It's clear that wrapping APIs should exist, but I'm not sure if this wrapping API should be created/maintained manually or autogenerated from the Windows API data.
* I may want to make it possible to generate subsets of the complete API.  One example of where this could be used it to generate bindings to be included in the standard library, where only a small portion of the complete API is required.

notnull.json
================================================================================
This file contains extra information about the win32 API, namely, which struct
fields and function parameters are "not null". By default, all pointer types in
the metadata are treated as being "optional" (i.e. `?*T`).  `notnull.json`
contains modifiers for types that make them non-optional (i.e. `*T`).
The format of `notnull.json` is the following:

```json
{
    "<api_name>": {
        "Functions": {
            "<function_name>": [<return_type_modifier>, <param_0_modifier>, <param_1_modifier>, ...]
            ...
        }
    }
    ...
}
```

The `return_type_modifier` and `param_X_modifier` values are integer flags.
Each flag in the integer modifies a pointer in the type to be "non-optional".
For example, consider the type `?*?*?*T`.  The integer value `0` would leave
the type unmodified.
The integer value `1` would modify the first pointer in the type to make it `*?*?*T`.
The integer value `2` would modify the second pointer in the type to become `?**?*T`.
The integer value `7` would modify all 3 pointer types to become `***T`.
