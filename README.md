# The perfect voxel engine SDK specification (v0.1)

The perfect voxel engine SDK for advanced game/sandbox developers. 

## Motivation

There are some good voxel engines out there, but those all seem to be very restrictive in terms of customization and come with large code bases that you need to understand and hack in order to add custom functionality. I want propose something more general in a form of API/SDK.

## Lanuage

Witten in C so can be used as header in HLSL/GLSL/C++/C and beyond.

## Functionality in pseudo code

`camera = createCamera(float viewDistance, float fov, etc)` // CPU only

`changeSettings(int gridChunkSize, etc)` // Global settings

`selectGridChunk(int x, int y, int z)` // Like world partition, but not just flat and why then name it WORLD

`changeViewerPosition(float x, float y, float z)` // Will start to stream grid chunk(s) dependent on position and view distance


`onGridChunkStream(int3 coords, voxel[] voxels)` // Event fired, whenever engine needs to stream new grid chunk from memory, or for procedural stuff

`selectGridRecArea(int x1, int y1, int z1, int x2, int y2, int z2)` // Rectangle selection, but other popular forms/brushes should exist

`registerMaterial(Color color, [other properties, like physics stuff])` // More properties, larger the map in MB

`setVoxelsInArea(material[x])` // Spawns voxel with given material in above selected area
 
 
`voxels = selectVoxelInRectArea(int x1, int y1, int z1, int x2, int y2, int z2)`

`group = groupVoxels(voxels)` // These voxels will be linked, related to each other trough group

`group = voxels[x].getGroup()`

`voxels = group.getVoxels()`



`transform(Voxel/Group voxel/group, pos, rot)` // Transform by force

`addForce((Voxel/Group voxel/group, float3 strength, float3 localPoint, ForceType force)` // Or with physics (force, impulse, acceleration)

`onForce(Voxel/Group voxel/group, force)` // Override/hack behavior



`addProperties(Voxel/Group voxel/group, [properties])` // Add more properties (like damage)

```
onCollision(Collider c1, Collider c2){ // Whenever 2 voxels collide
	impactPoint1 = c1.getPoint()
	velocity2 = c2.getVelocity()
	m1 = c1.getVoxel().getMaterial()
    // code the collision reaction your self, dependent on material etc
}
```

`rayCast(pos, dir, len)` // Most effective way possible

`load(str file, int3 gridChunkCoords)` // CPU only

`save(str file, int3 gridChunkCoords)` // CPU only

## What should happen under the hood

Engine takes care of acceleration structure creation and modification, compression, grid chunk stream in/out, LOD in most effective way. Provides also most effective ray cast (very important).

## Programmer will then take it from here

Programmer uses all of this to create own voxel game logic and experience. No Biased opinions on how physics should work, what renderer to use (taster/raytrace/pathtrace) and so on. The perfect engine SDK for voxel game developers.
