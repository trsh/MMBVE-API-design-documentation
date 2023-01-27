# MMBVE API design documentation (v0.11)

Minimalist multipurpose blocky voxel engine SDK API design documentation. 

## Motivation

There are some good voxel engines out there, but those all seem to be very restrictive in terms of customization and come with large code bases that you need to understand and hack in order to add custom functionality. I want to propose a minimalistic and generalized API that a voxel engine could provide/expose for advanced voxel game/sandbox developers.

## Lanuage

Witten in C so can be used as header in HLSL/GLSL/C++/C and beyond.

## Functionality in pseudo code

### Defentions

#### Camera:
```
Camera {
  float3 position,
  quat rotation,
  float viewDistance,
  float fov
}
```

- `viewDistance` engines uses this to calculate when to stream in/out partitions. If camera position + view distance is out of current partition (in any direction) and in next one, it should be streamed in. And via versa.
- `fov` field of view.
- `position` current position of camera
- `rotation` current rotation of camera

#### Partition:

Partitions are cubic shaped grids that form up into a larger uniform grid and are streamed in and out on demand. A partition where the player resides and it's neigbour partitions make up the space that player should be able to observe and where voxel manipulation is possible.

```
Partition {
  Voxel[] voxels,
  int3 position
}
```

- `position` x, y, z coordinates of the partition being streamed.
- `voxels` all voxels of the parition being streamed.

#### Voxel (WIP)

```
Voxel {
  // generic properties here
  float3 color,
  float3 velocity,
  float hardness
}
```
The data here can be generic. Whatever is needed for certain purpose. The structure will be the same for all voxels and has do be declared before runtime. // TODO: this needs more info

### Settings

`changeSettings(int gridPartitionSize, int minVoxelSize)` 

This should be called somwhere in game initialization phase to provide these important settings to the engine.

- `gridPartitionSize` is the size of a single partition. 
- `minVoxelSize` how small is the smallest voxel. This will have major impact on performance and visual appeal.

### Camera

`camera = createCamera(float viewDistance, float fov)` 

Defining the camera.

- `viewDistance` description in defintions.
- `fov` description in defintions.

`transformCamera(Camera camera, float3 position, quat rotation)` 

Transforming the camera. This can trigger streaming of partition(s) dependent on position and view distance.

- `camera` camera object
- `position` desired position of camera
- `rotation` desired rotation of camera

`onGridChunkStream(Partition partition)` 

Event fired whenever engine needs to stream (in/out) a partition. Can be used of procedural generation or other purposes.

- `partition` description in defintions.

### Selection

`selectGridPartition(int3 coords)` 

We select a one of the Streamed in paritions, to work with.

- `coords` partition global 3d coordinates

`selectGridRectAreaBegin(int3 start, int3 end)` 

- `start` star position in grid, or bottom left corner
- `end` end position in grid, or top right corner

//`registerMaterial(Color color, [other properties, like physics stuff])` // More properties, larger the map in MB

Rectangle selection, but other popular forms/brushes should exist. This does nothing on its own, but is needed for next function below.

`setVoxelsInArea(Voxel voxel)` 

Spawns block voxel with all its properties in selected area. 

- `Voxel` description in defintions.

`selectEnd()`

Cancel the selection. Its auto canceled when starting a new one.
 
`voxels = selectVoxelInRectArea(int3 start, int3 end)`

Get all voxels in rectangular area.

- `start` star position in grid, or bottom left corner
- `end` end position in grid, or top right corner

### Grouping

`Group group = groupVoxels(Voxel voxels = null)` 

Registers a voxel group. If the input value is null, it asumes that all voxels in active selection should be grouped.
This very usefull to represent sort of a Entity, like a rock, that is composed from multiple voxels.

`group = voxels[x].getGroup()`

To query of voxel is in a group.

`voxels = group.getVoxels()`

To query all voxels in a group.

### Transforms

`transform(Voxel/Group voxel/group, float3 position, float3 rotation)` 

Transform voxel or group by given position and rotation.

- `voxel/group` voxel or voxel group to transform
- `position` desired position for voxel/group
- `rotation` desired rotation for voxel/group

`onForce(Voxel/Group voxel/group, force)` // Override/hack behavior


### Updating

`addProperties(Voxel/Group voxel/group, [properties])` // TODO

Add or change properties of an existing voxel or group of voxels

- `voxel/group` voxel or voxel group for wich to change the properties
- `properties` TODO

### Collisions

```
onCollision(Collider c1, Collider c2){ 
	impactPoint1 = c1.getPoint()
	p1 = c1.getVoxel().[someProperty]
    	// code the collision reaction your self, dependent on properties etc.
}
```

Even fired whenever 2 voxels collide. 

### Raycast

`rayCast(float3 origin, float3 direction, float3 lenngth = null, uint lod = null)` 

Raycast voxels in streamed in partitions. Must be as efficient as possible, because will be probably used for rendering.

- `origin` origin of the ray
- `direction` direction to shoot the ray
- `lenngth` optional length for ray, by default its view distance
- `lodLevel` optional level of detail, by default is accepts leaf voxels. More coarse voxels from tree stracture can be fetched by providing depth level here (TODO)

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

Engine takes care of acceleration structure creation and modification, compression, partition stream in/out, raycast by lod, etc.

## Programmer will then take it from here

Programmer uses all of this to create own voxel game logic and experience. No Biased opinions on how physics should work, what renderer to use (taster/raytrace/pathtrace) and so on. 

### Questions and answers

#### Does this API provide rendering?

No. Not at all. You can use the Raycast for such purposes, but the rest is for you to implement.

#### Why do we need groups?

We can fast access voxels that make up some kind of a entity, like a door, rock, etc.

#### Ho can you rotate things in a grid structure?

I am not sure at the moment on the implemntation of this, but as you can see lots of voxe engines (Teardown, Atomontage, etc.) are doing this.

#### What is meant by locky?

Think of Minecraft, Teardown, etc. Blocks.


