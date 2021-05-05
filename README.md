# The Sponge Schematic Format Specification

The goal of the Sponge Schematic Format is to improve upon the previous [MCEdit Schematic Format](http://www.mcedit.net/) to provide better forward compatibility and improve inter compatibility
between different versions, platforms, and varyingly modded environments. This format may be used to serialize regions of a minecraft world to disk to be placed back into the world later. It
supports all types of modded blocks and block states as well as serializing Entities and TileEntities with the block data.

The format specification is NOT intended to represent schematics as objects for use in consumer applications, it is a specification for the *storage* of a schematic. As such, many object types are
specified to be as optimized as possible when used in storage.

## Current Version - 3

The current version of the Sponge Schematic Specification is 3 - and can be found [here](versions/schematic-3.md).

### Changelog

Version | Date | Changes
---|---|---
3 | 2021-05-04 | - Change support for 3D Biomes <br> - Clarify some wording on `varint` support
2 | 2019-05-08 | - Add Entities <br> - Add Biomes <br> - Add DataVersion per Minecraft versions <br> - Change `TileEntities` to `BlockEntities` for BlockEntity objects. <br> - Remove `ContentVersion` from various objects
1 | 2016-08-23 | Initial Version