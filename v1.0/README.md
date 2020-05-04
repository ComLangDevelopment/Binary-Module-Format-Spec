# Laser Language Binary Module Format

This format is derived from [LCS4: Binary IO](https://lightningcreations.github.io/LCS/publications/LCS4). 

Additionally, the meaning of "MUST", "SHALL", "MUST NOT", "SHALL NOT", "SHOULD", "SHOULD NOT", "MAY", "REQUIRED", "RECOMMENDED" and "OPTIONAL" are those specified by [[RFC 2119]](https://tools.ietf.org/html/rfc2119). 

Files produced in this format should either use the suffix .lso or .lsmod. 
Files in this format MUST NOT have the filename `$$`, 
Files of this format, and other formats 

## Module Format

```
struct laser_mod{
    u8 magic[4];
    version ver;
    u16 arch;
    constansts const_pool;
    u16 parent;
    u16 name;
    module_content contents;
    u32 crc;
};
```

arch shall have the a valid value for an Elf file's `e_machine` value as described in the Elf Specification. 

magic SHALL be the bytes `[7f 4c 13 12]`. ver SHALL be the version of the Laser Language Binary Module Format the file is produced for. 
The present version is `1.0` encoded as `[00 00]`.

parent shall be an index into `const_pool` which is a `Const_Utf8` which names the parent module of this module.
If this is the root module in a library or executable, the parent module shall be named `$$`.
name shall be an index into the `const_pool` which is a `Const_Utf8` which shall be one of the following:
* If the filename is a valid identifier that is neither $root, nor $mod, nor $$, ignoring any file suffix including and following the first ., name shall be that filename, ignoring any file suffix.
* If the filename is not a valid identifier, name may be any string. Such a file must not be accessed by a linker 
 codegenerator, or by a compiler for item name resolution. The file MUST be ignored by such a program. 
* If the filename is `$root`, then if the module is part of a lslib produced by compiling a bulb with some name *n* which is a valid identifier,
 then name shall be exactly *n*. Otherwise, if the module is part of an lslib produced by compiling any other bulb with some name *n*, 
 then name shall be an empty string. Otherwise, name may either be an empty string or any valid identifier except for `$$`.
* If the filename is `$mod`, and is part of some directory with name *m*, then name shall be exactly *m* (*m* must be a valid identifier). 
* If the filename is `$$`, then any linker, code generator, or compiler MUST reject this file and any lslib which contains this file.
* If no filename component is available (IE. the file is further encoded in such a method that does not include or produce filename infomration), 
 it is implementation-defined what names are permitted for this module. It is implementation-defined whether the file may be loaded by linkers,
 code generators, or compilers. 

crc shall be a CRC32 applied with the polynomial 0x04C11DB7 over the contents of the file. 

### Constant Pool

```
struct constants{
    u16 len;
    constant entries[len];
};

struct constant{
     u8 tag;
     union{
          string Const_Utf8;
          struct {u16 len; u8 bytes[len];} Const_bstring;
          u32    Const_u32;
          u64    Const_u64;
          i32    Const_i32;
          i64    Const_i64;
          u128   Const_u128;
          i128   Const_i128;
          f32    Const_f32;
          f64    Const_f64;
          path   Const_qIdent;
          type Const_type;	
    };
};
```

tag shall be one of `Const_Utf8` (0), `Const_bstring` (1), `Const_u32` (2), `Const_u64 (3)`, `Const_i32` (4), `Const_i64` (5),
 `Const_u128` (6), `Const_i128` (7), `Const_f32` (8), `Const_f64` (9), `Const_qIdent` (10), or `Const_type` (11).

```
struct path{
    u8 n_components;
    u16 components[n_components];
};
```

Each item in components shall be a valid index into the constant pool which is a `Const_Utf8` that represents a valid identifier. 
If any component represents the string `$$`,`$root`, or `$mod`, that component must be the first item. 

```
struct type{
    u8 tag;
    union{
        u16 t_named;
        u8 t_generic;
        u16 t_slice;
        struct { u16 type; u64 size;} t_array;
        struct { u16 type; u8 generic_item;} t_generic_array;
        struct {
               u16 return;
               u8 n_generic_types;
               u8 n_generic_vals;
               u16 generic_vals_types[n_generic_vals];
               u8 n_captures;
               u16 capture_types[n_captures];
               u8 n_params;
               u16 param_types[n_params];
      } t_function;
      u16 t_ref;
      u16 t_mut_ref;
      u16 t_const_ptr;
      u16 t_mut_ptr;
      struct { u8 n_types; u16 types[n_types];} t_tuple;
    };
};
```

tag shall be `t_i8` (0), `t_i16` (1), `t_i32` (2), `t_i64` (3), `t_i128` (4), `t_isize` (5), `t_u8` (6), `t_u16` (7),
 `t_u32` (8), `t_u64` (9), `t_u128` (10), `t_usize` (11), `t_generic`(12), `t_slice` (13), `t_array` (14), `t_generic_array` (15), 
 `t_ref` (16), `t_mut_ref` (17), `t_const_ptr` (18), `t_mut_ptr` (19), or `t_tuple` (20). 


  


