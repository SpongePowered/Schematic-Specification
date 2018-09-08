**This specification is under development**
# Sponge Schematic Specification

**Version 2**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Introduction

This specification defines a format which describes an area of a [Minecraft](http://minecraft.net) world for the purpose of serialization to disk and deserialization from disk. It is designed in order to allow maximum cross-compatibility between platforms, versions, and various states of modification.

## Revision History

Version | Date | Notes
---|:---:|---
1 | 2016-08-23 | Initial Version
2 | 2018-08-19 | Extendsion in progress

## Specification

### Format

All field names in the specification are **case sensitive**.

#### <a name="formFile"></a>File

The structure described by this specification is persisted to disk using the [Named Binary Tag](http://minecraft.gamepedia.com/NBT_format) (NBT) format (also see [here](https://wiki.vg/NBT)). Before writing to disk the NBT data must be compressed using the [GZip](https://www.gnu.org/software/gzip/) data compression algorithm. The highly recommended file extension for files using this specification is `.schem` (chosen so as to not conflict with the legacy `.schematic` format allowing easy distinction between the two).

#### <a name="formId"></a>Identification string

Resources identified by a resource location have an identifier string which is made up of two sections, the domain and the name, written as `domain:name`. If the domain is not specified then it is assumed to be `minecraft`. For example `planks` would identify the same resource as `minecraft:planks` as the `minecraft:` prefix is implicit.

### Schema

#### <a name="schematicObject"></a>Schematic Object

This is the nameless root object of the schematic. It shall be a [NBT](#formFile) tag compound.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="schematicVersion"></a>Version | `NBT int` | **Required.** Specified the format version being used. It may be used to provide validation and auto-conversion from older versions. The current version is `2`. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicContentVersion"></a>ContentVersion | `NBT int` | **Required.** A version identifier for the contents. Representing the minecraft version, when serialized. Used for providing backwards compatibility. [Values are specified by Mojang](https://minecraft-de.gamepedia.com/Versionen#Version-IDs). Mojangs data converter for older versions will need this value for proper conversion. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicMetadata"></a>Metadata | [Metadata Object](#metadataObject) as `NBT compound` | **Optional** Provides metadata about the schematic. This shall be saved as [NBT](#formFile) tag compound.
<a name="schematicWidth"></a>Width | `NBT int` | **Required.** Specifies the width (the amount of blocks along the X-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicHeight"></a>Height | `NBT int` | **Required.** Specifies the height (the amount of blocks along the Y-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicLength"></a>Length | `NBT int` | **Required.** Specifies the length (the amount of blocks along the Z-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicOffset"></a>Offset | `NBT int array`[3] | **Optional** Specifies the relative offset of the schematic. This value is relative to the most minimal point in the schematics area. If present, this offset will be added to all blocks, tiles and entities when pasting it into the world (may be negative). This shall be saved as [NBT](#formFile) tag int array.
<a name="schematicPaletteMax"></a>PaletteMax | `NBT byte` | **Required** Specifies the size of the block palette index in number of bytes needed for the maximum palette index. Implementations must use this to determine, which index format will be needed. The palette index may fit within a datatype smaller than a 32-bit integer or even a longer datatype like 64-bit integer. This shall be saved as [NBT](#formFile) tag byte.
<a name="schematicPalette"></a>Palette | `NBT list` of [Palette Object](#paletteObject) | **Required** Specifies the block palette. This is a mapping of block states to indices which are local to this schematic. The array element indices are used to reference the block states from within the [BlockData array](#schematicBlockData). The maximum index cannot be greater than [`2^(PaletteMax * 8) - 1`](#schematicPaletteMax).  This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag string.
<a name="schematicBlockData"></a>BlockData | `nbt int array` or `nbt long array` | **Required.** Specifies the main storage array which contains `Width * Height * Length` entries. Each entry is specified as a `int` or `long`, depending on the given [PaletteMax](#schematicPaletteMax) (1 to 4 bytes = int; 5 to 8 bytes = long). This index refers to the array element index within the [Palette](#schematicPalette). The entries are indexed by `x + z * Width + y * Width * Length`. This shall be saved as [NBT](#formFile) tag int array or [NBT](#formFile) tag long array.
<a name="schematicTileEntities"></a>TileEntities | `NBT list` of [TileEntity Object](#tileEntityObject) | **OPTIONAL** Specifies additional data for blocks which require extra data. If no additional data is provided for a block which normally requires extra data then it is assumed that the TileEntity for the block is initialized to its default state. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag compound.
<a name="schematicEntities"></a>Entities | `NBT list` of [Entity Object](#entityObject) | **OPTIONAL** Specifies all entities, within the area of this schematic. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag compound.


#### <a name="metadataObject"></a>Metadata Object

An object which provides optional additional meta information about the schematic. The fields outlined here are guidelines to assist with standardization but it is recommended that any program reading and writing schematics persist all fields found within this object.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="metadataName"></a>Name | `string` | **OPTIONAL** The name of the schematic. This shall be saved as [NBT](#formFile) tag string.
<a name="metadataAuthor"></a>Author | `string` | **OPTIONAL** The name of the author of the schematic. This shall be saved as [NBT](#formFile) tag string.
<a name="metadataDate"></a>Date | `long` | **OPTIONAL** The date that this schematic was created on. This is specified as seconds since the Unix epoch. This shall be saved as [NBT](#formFile) tag long.
<a name="metadataRequiredMods"></a>RequiredMods | `string`[] | **OPTIONAL** An array of mod ids which have blocks which are referenced by this schematic's defined [Palette](#schematicPalette). May be empty. This shall be saved as [NBT](#formFile) tag list of NBT tag string.

##### Metadata Object Example:

```js
{
    "Name": "My Schematic",
    "Author": "Author Name",
    "Date": 1534701119,
    "RequiredMods": [
        "a_mod",
        "another_mod"
    ]
}
```

#### <a name="paletteObject"></a>Palette Object

An object which holds a mapping of a block state id to an index. The indices are given by the array index and won't be greater than [`2^(PaletteMax * 8) - 1`](#schematicPaletteMax).
This object shall be a [NBT](#formFile) tag list of [NBT](#formFile) tag string.

##### Block State

A Block State is an instance of a block type with a set of extra data used to further define the material. The additional data varies by block type and a complete listing of vanilla types is available [here](http://minecraft.gamepedia.com/Block_states).

The format of the Block State identifier is the [ID](#formId) of the block type and a set of comma-separated property `key=value` pairs surrounded by square brackets. If the block has no properties then they can be excluded. The block type [ID](#formId) is specified as a resource location.

For example the air block has no properties so its representation would be just the block type [ID](#formId) `minecraft:air`. The planks block however has an enum property for the `variant` so its identification would be `minecraft:planks[variant=oak]`. Properties may be ordered with their keys in alphabetical ordering.

##### Palette Object Example

```js
"Palette" [
    "minecraft:air",
    "minecraft:planks[variant=oak]",
    "a_mod:custom"
]
```

#### <a name="tileEntityObject"></a>Tile entity object

An object to specify a tile entity which is within the area. Tile entities are used by Minecraft to store additional data for a block (such as the lines of text on a sign or contents of a chest). The fields used to describe a tile entity vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="tileEntityPos"></a>Pos | `unsigned integer[3]` | **Required.** The position of the tile entity relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffset) applied). Must contain exactly 3 integer values. This shall be saved as [NBT](#formFile) tag int array.
<a name="tileEntityId"></a>Id | `string` | **Required.** The id of the tile entity type defined by this tile entity object, specified as a resource location. This should be used to identify which fields should be required for the definition of this type. This shall be saved as [NBT](#formFile) tag string.
<a name="tileEntityProp"></a>{additional properties} | `<?>` | Amount and format of additional fields depending on the tile entity.

##### Tile Entity Object Example

An example of possible storage of a sign. See the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format) for a complete listing of data used to store various types of tile entities present in vanilla minecraft. Mods may store additional data or have additional types of tile entities.

```js
{
    "ContentVersion": 0,
    "Pos": [0, 1, 0],
    "Id": "minecraft:Sign",
    "Text1": "foo",
    "Text2": "",
    "Text3": "bar",
    "Text4": ""
}
```

#### <a name="entityObject"></a>Entity Object

An object to specify all entitys which are within the area. Entities are used by Minecraft for movable or interactable objects (such as mobs or minecarts). The fields used to describe a entity vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="entityPos"></a>Pos | `double[3]` | **Required.** The position of the entity relative to the `[0.0, 0.0, 0.0]` position of the schematic (without the [offset](#schematicOffset) applied). Must contain exactly 3 double values. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag double.
<a name="entityId"></a>Id | `string` | **Required.** The id of the entity type, specified as a resource location. This should be used to identify which fields should be required for the definition of this type. This shall be saved as [NBT](#formFile) tag string.
<a name="entityProp"></a>{additional properties} | `<?>` | Amount and format of additional fields depending on the entity.

##### Entity Object Example

An example of possible storage of a sign. See the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format) for a complete listing of data used to store various types of tile entities present in vanilla minecraft. Mods may store additional data or have additional types of entities.

```js
{
    "ContentVersion": 0,
    "Pos": [0, 1, 0],
    "Id": "minecraft:Sign",
    "Text1": "foo",
    "Text2": "",
    "Text3": "bar",
    "Text4": ""
}
```

## Changelog

Following changes has been made compared to version 1:

### Functional changes

* Changed [Palette](#schematicPalette) from `object` with key as ids and value as index to a `list` with ids, so you can easily calculate an index and **access it directly without searching**.
* Changed [BlockData](#schematicBlockData) from `varint[]` to `int[]` or `long[]` to support even bigger areas and because **NBT has no varint or varlong definition**.
* Changed [Width](#schematicWidth), [Height](#schematicHeight), [Length](#schematicLength) from `unsigned short` to `unsigned integer` to support even **bigger areas**.
* Changed [PaletteMax](#schematicPaletteMax) from `integer` to `byte`, because **8 would be the maximum** possible value for Java implementations (64-bit long)
* Added [Entity](#entityObject) field to **support entities**.
* Changed [Date](#metadataDate) from miliseconds to seconds to the default **UNIX timestamp**, to cover a longer time range and because noone needs miliseconds.
* Moved [ContentVersion](#schematicContentVersion) from entity & tile compounds up to the root compound. Because there is only one relevant version when exported, there is **no need for versioning each tile & entity**.

### Other changes

* Added **OPTIONAL** or **REQUIRED** in every field to make it more clear
* Replaced the word `region` by the word `area`, to avoid misunderstanding since a minecraft world already has so called `regions`
* Made the offset more clear
* Moved Definition#BlockState to chapter PaletteObject
* Added the corresponding NBT type to every field
