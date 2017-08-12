# Open-Source Binary Model Format

This document describes the specification for version 1 of the SEModel file format.

## Basic Data Types
The following types are used through this specification file.

```c++
//
// Fundamental data types
//
typedef unsigned char	uint8_t;	// 1 byte unsigned integer
typedef unsigned short	uint16_t;	// 2 byte unsigned integer
typedef unsigned long	uint32_t;	// 4 byte unsigned integer

//
// string_t is used in this document to  describe a null terminated array of characters (a string)
//
typedef char string_t[];

#if dataPropertyFlags & SEANIM_PRECISION_HIGH
	typedef double vec3[3];
	typedef double quat_t[4]; // X Y Z W (Normalized)
#else
	typedef float vec3[3];
	typedef float quat_t[4]; // X Y Z W (Normalized)
#endif

//
// bone_t data type is used to describe bone indicies, its size is automatically determined by the number of bones in the model
//
#if boneCount <= 0xFF
	typedef uint8_t bone_t;
#elif boneCount <= 0xFFFF
	typedef uint16_t bone_t;
#else //elif boneCount <= 0xFFFFFFFF
	typedef uint32_t bone_t;
#endif
```
---
## SEModel File Structure
The general structure of a *.semodel file conists of a 7 byte magic value containing the characters 'SEModel' followed by a 16bit version identifier, the file [header](#seanim-header), and then the data blocks.
The data blocks must follow the order defined below; Although each of these data blocks is optional, there must be *at least* 1 data block (excluding the custom data block) present within a given file. (See [here](#seanim-data-flags) for more information on how to describe the presence of each of these data blocks).