# MMBVE API design documentation (v0.3)

Minimalist multipurpose blocky voxel engine API design documentation. With blocky we mean Minecraft/Teardown style.

## Motivation

There are some good voxel engines out there, but those all seem to be very restrictive in terms of customization and come with large code bases that you need to understand and hack in order to add custom functionality. I want to propose a minimalistic and generalized API that a voxel engine could provide/expose for advanced voxel game/sandbox developers.

## Lanuage

Ideally witten in C so can be used as header in HLSL/GLSL/C++/C and beyond (inspired by https://github.com/AcademySoftwareFoundation/openvdb/blob/master/nanovdb/nanovdb/PNanoVDB.h). Neverless some parts belong to CPU only, for othes it depends on the implementation. The tehnical imlementation details are out of the scope of this document.

## Functionality in pseudo code

### Defentions of structures (only public properties)

#### Camera
```
Camera {
  float3 position,
  float3 rotation,
  float viewDistance,
  float fov,
  int3 partitionCoords
}
```

- `viewDistance` engines uses this to calculate when to stream in/out partitions. If camera position + view distance is out of current partition (in any direction) and in next one, it should be streamed in. And via versa.
- `fov` field of view.
- `position` current position of camera
- `rotation` current rotation of camera in euler angles
- `partitionCoords` partition coordinates where camera resides at this moment

#### Partition

Top view for illustration purposes only:  
![vxp](https://user-images.githubusercontent.com/3727523/215129017-43bc99f0-9022-4e8e-8630-94697078bd7f.png)


Partitions are cubic shaped grids that form up into a larger uniform grid and are streamed in and out on demand (think of Unreal World partitions, but 3D). A partition where the player resides and it's neigbour partitions make up the space that player should be able to observe and where voxel manipulations/events are possible. The goal is to always have 26 neighbour partitions around current one.

```
Partition {
  int3 position
}
```

- `position` x, y, z coordinates of the partition.

#### Voxel

```
Voxel {
  // generic properties here - primitives only
  float3 color,  
  float hardness
}
```
The data here is generic. Whatever is needed for certain purpose. The structure will be the same for all voxels and has do be declared before runtime. More properties, larger the map in MB. All possible varinations have to be registered (see in fallowing section "Registers").

#### Group

```
Group {
  // generic properties here - primitives only
  float damage,
  float3 velocity
}
```

The data here is generic. Whatever is needed for certain purpose. The structure will be the same for all groups and has do be declared before runtime. More properties, larger the map in MB.


#### Voxel accessor

```
VoxelAccessor {
  int3 position,
  int lod,
  Voxel voxel
}
```

- `position` voxel index position in the grid
- `lod` depth of voxel
- `voxel` structure described above

This is object you use for updating and querying voxel.

#### Voxels selector

```
VoxelsSelector {
  VoxelAccessor[] getAll(),
  VoxelAccessor[] slice(uint from, uint to),
  VoxelAccessor[] splice(uint from, uint to, VoxelAccessor va = null),
}
```

- `getAll()` getter for all voxel accessors
- `slice(uint from, uint to)` take a piece of voxel accessors
- `splice(uint from, uint to, VoxelAccessor va = null)` insert, replace or remove voxel accessor(s) // concept from [JavaScript splice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)

This is object is used to point to multiple voxels.

#### Group accessor

```
GroupAccessor {
  float3 position,
  float3 rotation,
  int voxelCount,
  Group group
}
```

- `position` current position of group
- `rotation` current rotation of group in euler angles
- `voxelCount` total voxel count in group
- `group` structure described above

This is object you use for updating and querying group.

#### Collider

```
Collider {
  impactPoint,
  voxelAccessor
}
```

Object holding info about collision.

- `impactPoint` local hit point
- `voxelAccessor` voxel colliding

#### Hit

```
Hit {
  float3 localPointOnVoxel
  VoxelAccessor voxel
  float length
}
```

- `localPointOnVoxel` local hit point
- `voxel` voxel being hit
- `length` ray total length



### Registers

Engine can make impressive optimziations if it knows all possible variations of the voxel generic data. There for we have function:

`registerVoxelProperties<T = propery of Voxel>(keyof T propery, valueof T[] values)`

- `propery` generic property like `color`
- `values` generic property value like `{ float3(1,0,0), float3(1,1,0) }`

NOTE: This is not optional.

### Settings

`changeSettings(int gridPartitionSize, int minVoxelSize, int maxTreeDepth, Enum accelerationStructure = null)` 

This should be called somwhere in game initialization phase to provide these important settings to the engine.

- `gridPartitionSize` is the size of a single partition
- `minVoxelSize` how small is the smallest voxel. This will have major impact on performance and visual appeal
- `maxTreeDepth` how many lod levels each partition grid tree (acceleration structure) will have
- `accelerationStructure` engine can provide multiple acceleration structures, like octees, brickmaps, sparse brick sets, etc. If you change this after saving a map, you wont be able, most likely to load it, unless there is some centralized storage format.

### Camera

`camera = createCamera(float viewDistance, float fov)` 

Defining the camera.

- `viewDistance` description in defintions
- `fov` description in defintions

`transformCamera(Camera camera, float3 position, float3 rotation)` 

Transforming the camera. This can trigger streaming of partition(s) dependent on position and view distance.

- `camera` camera object
- `position` desired position of camera
- `rotation` desired rotation of camera in euler angles

`onGridChunkStream(Partition partition)` 

Event fired whenever engine needs to stream (in/out) a partition. Can be used of procedural generation or other purposes.

- `partition` description in defintions

`Partition[] partitions = getAllActivePartitions(int3 coords = null)` 

Get all stramed in or streaming in paritions at this moment.

- `coords` if provided, will return 1 respestive partition or null if its not active

### Selection and spawning

`selectPartition(int3 coords)` 

We select a one of the Streamed in paritions, to work with. All the things below will take place in this selected partition and its local coordinates. It should throw error, if parition is not active streamed in.

- `coords` partition global 3d coordinates

`VoxelsSelector voxelsSel = setVoxels(int3[] gridCellCoords, Voxel voxel)` 

Spawns block voxel with all its properties in selected area, all of it. Returns those new voxels.

- `gridCellCoords` grid cell coordinates where to spawn voxel
- `voxel` description in defintions
 
`VoxelsSelector voxelsSel = selectVoxelInRectArea(int3 start, int3 end)`

Get all voxels in rectangular area.

- `start` star position in grid, or bottom left corner
- `end` end position in grid, or top right corner


### Grouping

`GroupAccessor group = groupVoxels(VoxelsSelector voxelsSel)` 

Registers a voxel group. This very usefull to represent sort of a Entity, like a rock, that is composed from multiple voxels.

Group can never be larger than one parition.

`GroupAccessor group = getVoxelGroup(VoxelAccessor voxel)`

Check/get if voxel is in a group.

`VoxelsSelector voxelsSel = getGroupVoxels(GroupAccessor group)`

To get all voxels in a group. Voxels belonging to a partition that is not yet streamed in wont be returned. This can be checked by 'GroupAccessor.voxelCount'

### Transforms

`transform(GroupAccessor group, float3 position, float3 rotation)` 

Transform group by given position and rotation.

- `group` voxel group accessor
- `position` desired position for group
- `rotation` desired rotation for group in euler angles

### Updating

`changeProperty<T = propery of Voxel>(VoxelAccessor voxel, keyof T property, valueof T value)`\
`changeProperty<T = propery of Voxel>(VoxelsSelector voxelSel, keyof T property, valueof T value)`\
`changeProperty<T = propery of Group>(GroupAccessor group, keyof T property, valueof T value)`

Change properties of an existing voxel or group of voxels

- `voxels/voxelSel/group` voxel accessor, voxel selection or voxel group accessor
- `property` generic property
- `value` generic property value

### Collisions

```
onCollision(Collider c1, Collider c2){  // Colliders for both voxels, description is in defintions
	float3 impactPoint1 = c1.impactPoint
	VoxelAccessor va2 = c2.voxelAccessor
    	// code the collision reaction your self, dependent on properties etc.
}
```

Event fired whenever 2 voxels collide. 

### Raycast

`Hit hit = rayCast(float3 origin, float3 direction, float3 lenngth = null, uint lod = null)` 

Raycast voxels in streamed in partitions. Must be as efficient as possible, because will be probably used for rendering.

- `origin` origin of the ray
- `direction` direction to shoot the ray
- `lenngth` optional length for ray, by default its view distance
- `lodLevel` optional level of detail, by default is accepts leaf voxels. More coarse voxels from tree stracture can be fetched by providing depth level here. Its properties are interpolated from child voxels. 

### Load and store

`save(str file, int3 partitionCoords)`

Store parition. 

- `file` file destination
- `partitionCoords` partition coords to store

`load(str file, int3 partitionCoords)` 

Manually load some parition.

- `file` file destination
- `partitionCoords` partition coords to load

## What should happen under the hood

Engine takes care of acceleration structure creation and modification, compression, partition stream in/out, raycast by lod, etc. Also speculatively deactivates and activates colliders depenging on actions in the parition.

## Programmer will then take it from here

Programmer uses all of this to create own voxel game logic and experience. No Biased opinions on how physics should work, what renderer to use (taster/raytrace/pathtrace) and so on. 

## Questions and answers

#### Does this API provide rendering?

No. Not at all. You can use the Raycast for such purposes, but the rest is for you to implement.

#### Does this API advanced physics?

No. Not at all. Only collisions, but the rest is for you to implement.

#### Why do we need groups?

We can fast access voxels that make up some kind of a entity, like a door, rock, etc.

#### How can you rotate and transform groups in a grid structure with float precision?

As you can see lots of voxe engines (Teardown, The Sandbox, Atomontage, etc.) are already doing this. Thencial details are not part of this document.

#### What is meant with 'blocky'?

Think of Minecraft, Teardown, etc. Blocks.

#### Why do I need to deal with paritions?

It is essential for procedural generation. And also for coordination system. Each partition has local one. If we use global coordinates, the numbers can get insanely large to impossible. 

