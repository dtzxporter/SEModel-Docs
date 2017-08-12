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
typedef signed long 	int32_t;	// 4 byte signed integer

//
// Half-precision data types
//
typedef unsigned short half_t;		// 2 byte floating-point data (IEEE 754-2008)

//
// string_t is used in this document to  describe a null terminated array of characters (a string)
//
typedef char string_t[];

//
// vec and quat types are used in this document to describe vertex and bone data, for this reason, we don't support high precision, due to the size that it would impose on the buffers with little to no benifit
//
typedef float vec2[2];
typedef float vec3[3];
typedef float quat_t[4]; // X Y Z W (Normalized)

//
// half vec types are used for data that doesn't require such high precision as a normal floating point number
//
typedef half_t vec2_h[2];
typedef half_t vec3_h[3];

//
// byte vec types are used for color data and any small precision data
//
typedef unsigned char vec3_b[3];
typedef unsigned char vec4_b[4];


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

//
// mat_t data type is used to describe material indicies, its size is automatically determined by the number of materials in the model
//
#if matCount <= 0xFF
	typedef uint8_t mat_t;
#else
	typedef uint16_t mat_t;
#endif
```
---
## SEModel File Structure
The general structure of a *.semodel file conists of a 7 byte magic value containing the characters 'SEModel' followed by a 16bit version identifier, the file [header](#semodel-header), and then the data blocks.
The data blocks must follow the order defined below; Although each of these data blocks is optional, there must be *at least* 1 data block (excluding the custom data block) present within a given file. (See [here](#semodel-data-flags) for more information on how to describe the presence of each of these data blocks).
```c++
struct SEModel_File
{
	char magic[7];			// 'SEModel'
	uint16_t version;		// The file version - the current version is 0x1
	SEModel_Header header;

	if (header.dataPresenceFlags & SEMODEL_PRESENCE_BONE)
	{
		string_t bone[header.boneCount];
		SEModel_BoneData[header.boneCount];
	}

	if (header.dataPresenceFlags & SEMODEL_PRESENCE_MESH)
	{
		SEModel_MeshData[header.meshCount];
	}

	if (header.dataPresenceFlags & SEMODEL_PRESENCE_COLLISION)
	{
		// TODO: Implement collision data
	}

	if (header.dataPresenceFlags & SEMODEL_PRESENCE_MATERIALS)
	{
		// TODO: Implement material data
	}
}
```

## SEModel_Header
The following defines the structure for the header structure of a SEModel file. Any reserved fields should be set to 0.
```c++
struct SEModel_Header
{
	// Contains the size of the header block in bytes, any extra data is ignored
	uint16_t headerSize; // Currently 0xDEADBEEF

	// Model type flags, matches SEMODEL_TYPE
	uint8_t modelType;

	// Model flags, implementation defined
	uint16_t modelFlags;

	// Bitwise flags that define which data blocks are present, matches SEMODEL_PRESENCE_FLAGS enum
	uint8_t dataPresenceFlags;

	// Bitwise flags that define which data is present in the boneData block, matches SEMODEL_BONEPRESENCE_FLAGS enum
	uint8_t boneDataPresenceFlags;

	// Bitwise flags that define which data is present in the meshData blocks, matches SEMODEL_MESHPRESENCE_FLAGS enum
	uint8_t meshDataPresenceFlags;

	// RESERVED - Should be 0
	uint8_t reserved1[2];

	// Is 0 if ( dataPresenceFlags & SEANIM_PRESENCE_BONE ) is false
	uint32_t boneCount;
	// Is 0 if ( dataPresenceFlags & SEMODEL_PRESENCE_MESH ) is false
	uint32_t meshCount;
	// Is 0 if ( dataPresenceFlags & SEMODEL_PRESENCE_COLLISION ) is false
	uint32_t colCount;
	// Is 0 if ( dataPresenceFlags & SEMODEL_PRESENCE_MATERIALS ) is false
	uint32_t matCount;

	// Is 0 if ( dataPresenceFlags & SEMODEL_PRESENCE_MESH ) is false or if ( meshDataPresenceFlags & SEMODEL_PRESENCE_WEIGHTS ) is false
	uint8_t maxSkinInfluence;

	// RESERVED - Should be 0
	uint8_t reserved2[3];
}
```
## SEMODEL_TYPE
The following is the recommended values for modelType (SEMODEL_TYPE), note that this may be implementation defined and is not used for importing. See boneDataPresenceFlags to specify whether or not the model has bones.
```c++
enum SEMODEL_TYPE
{
	SEMODEL_TYPE_STATIC,
	SEMODEL_TYPE_DYNAMIC,
	SEMODEL_TYPE_COLLISION
}
```
---

## SEMODEL_PRESENCE_FLAGS
SEMODEL_PRESENCE_FLAGS describes the bitfields used by dataPresenceFlags in the [header](#semodel-header). It only contains information pertaining to the presence of types of data included in the file.
```c++
enum SEMODEL_PRESENCE_FLAGS
{
	// Whether or not this model contains a bone block
	SEMODEL_PRESENCE_BONE = 1 << 0,
	// Whether or not this model contains submesh blocks
	SEMODEL_PRESENCE_MESH = 1 << 1,
	// Whether or not this model contains collision data blocks
	SEMODEL_PRESENCE_COLLISION = 1 << 2,

	// Whether or not this model contains a material reference block
	SEMODEL_PRESENCE_MATERIALS = 1 << 3,
	
	// RESERVED_0		= 1 << 4, // ALWAYS FALSE
	// RESERVED_1		= 1 << 5, // ALWAYS FALSE
	// RESERVED_2		= 1 << 6, // ALWAYS FALSE
	// RESERVED_3		= 1 << 7, // ALWAYS FALSE
}
```
---

## SEMODEL_BONEPRESENCE_FLAGS
SEMODEL_BONEPRESENCE_FLAGS describes the bitfields used by boneDataPresenceFlags in the [header](#semodel-header). It only contains information pertaining to the presence of types of data included in the bone block.
```c++
enum SEMODEL_BONEPRESENCE_FLAGS
{
	// Whether or not bones contain global matricies
	SEMODEL_PRESENCE_GLOBAL_MATRIX = 1 << 0,
	// Whether or not bones contain local matricies
	SEMODEL_PRESENCE_LOCAL_MATRIX = 1 << 1,

	// Whether or not bones contain scales
	SEMODEL_PRESENCE_SCALES = 1 << 2,
	
	// RESERVED_0		= 1 << 3, // ALWAYS FALSE
	// RESERVED_1		= 1 << 4, // ALWAYS FALSE
	// RESERVED_2		= 1 << 5, // ALWAYS FALSE
	// RESERVED_3		= 1 << 6, // ALWAYS FALSE
	// RESERVED_4		= 1 << 7, // ALWAYS FALSE
}
```
---

## SEMODEL_MESHPRESENCE_FLAGS
SEMODEL_MESHPRESENCE_FLAGS describes the bitfields used by meshDataPresenceFlags in the [header](#semodel-header). It only contains information pertaining to the presence of types of data included in the mesh vertex blocks.
```c++
enum SEMODEL_MESHPRESENCE_FLAGS
{
	// Whether or not meshes contain at least 1 uv layer
	SEMODEL_PRESENCE_UVSET = 1 << 0,

	// Whether or not meshes contain vertex normals
	SEMODEL_PRESENCE_NORMALS = 1 << 1,

	// Whether or not meshes contain vertex colors
	SEMODEL_PRESENCE_COLOR = 1 << 2,

	// Whether or not meshes contain at least 1 weight set
	SEMODEL_PRESENCE_WEIGHTS = 1 << 3,
	
	// RESERVED_0		= 1 << 4, // ALWAYS FALSE
	// RESERVED_1		= 1 << 5, // ALWAYS FALSE
	// RESERVED_2		= 1 << 6, // ALWAYS FALSE
	// RESERVED_3		= 1 << 7, // ALWAYS FALSE
}
```
---

## SEModel_BoneData
```c++
struct SEModel_BoneData
{
	uint8_t flags;

	int32_t boneParent; // Is -1 if bone is a root bone, otherwise, index of parent bone.

	if (header.boneDataPresenceFlags & SEMODEL_PRESENCE_GLOBAL_MATRIX)
	{
		vec3 global_position;
		quat_t global_rotation;
	}

	if (header.boneDataPresenceFlags & SEMODEL_PRESENCE_LOCAL_MATRIX)
	{
		vec3 local_position;
		quat_t local_rotation;
	}

	if (header.boneDataPresenceFlags & SEMODEL_PRESENCE_SCALES)
	{
		vec3 scale;	// 1.0 is the default scale, 2.0 is twice as big, and 0.5 is half size
	}
}
```
---

## SEModel_MeshData
```c++
struct SEModel_MeshData
{
	uint8_t flags;

	// The count of UV sets per vertex in this mesh
	uint8_t uvSetCount;

	// The count of verticies in the mesh
	uint32_t vertexCount;
	// The count of faces in the mesh, faces match D3DPT_TRIANGLELIST (DirectX) and VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST (Vulkan)
	uint32_t faceCount;

	SEModel_VertexData[vertexCount];
	SEModel_FaceData[faceCount];
}
```
---

## SEModel_VertexData
```c++
struct SEModel_VertexData
{
	vec3 position;

	if (header.meshDataPresenceFlags & SEMODEL_PRESENCE_UVSET)
	{
		vec2 uvCoords[mesh.uvSetCount];
		mat_t uvMaterialIndicies[mesh.uvSetCount];
	}

	if (header.meshDataPresenceFlags & SEMODEL_PRESENCE_NORMALS)
	{
		vec3_h normal;
	}

	if (header.meshDataPresenceFlags & SEMODEL_PRESENCE_COLOR)
	{
		vec4_b color;
	}

	if (header.meshDataPresenceFlags & SEMODEL_PRESENCE_WEIGHTS)
	{
		bone_t boneIndex[header.maxSkinInfluence];
		half_t boneWeight[header.maxSkinInfluence];
	}
}
```
---

## SEModel_FaceData
```c++
struct SEModel_FaceData
{
	uint32_t Indicies[3];
}
```