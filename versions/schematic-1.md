# Sponge Schematic Specification

#### Version 1

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Introduction

This specification defines a format which describes a region of a [Minecraft](http://minecraft.net) world for the purpose of serialization and storage of the region to disk. It is designed in order to allow maximum cross-compatibility between platforms, versions, and various states of modding.

## Revision History

Version | Date | Notes
--- | --- | ---
1.0 | 2016-MM-DD | Initial release of the specification

## Definitions

##### <a name="defBlockState"></a>Block State
A Block State is an instance of a block type with a set of extra data used to further define the material. The additional data varies by block type and a complete listing of vanilla types is available [here](http://minecraft.gamepedia.com/Block_states).

## Specification

### Format

The the structure described by this specification is persisted to disk using the [Named Binary Tag](http://minecraft.gamepedia.com/NBT_format) (NBT) format. Before writing to disk the NBT data must be compressed using the [GZip](https://www.gnu.org/software/gzip/) data compression algorithm.

All field names in the specification are **case sensitive**.

### Schema

#### <a name="schematicObject"></a>Schematic Object

This is the root object for the specification.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="schematicVersion"></a>Version | `integer` | **Required.** Specified the format version being used. It may be used to provide validation and auto-conversion from older versions. The current version is `1`.
<a name="schematicMetadata"></a>Metadata | [Metadata Object](#metadataObject) | Provides optional metadata about the schematic.
<a name="schematicWidth"></a>Width | `unsigned short` | **Required.** Specifies the width (the size of the area in the X-axis) of the schamatic.
<a name="schematicHeight"></a>Height | `unsigned short` | **Required.** Specifies the height (the size of the area in the Y-axis) of the schamatic.
<a name="schematicLength"></a>Length | `unsigned short` | **Required.** Specifies the length (the size of the area in the Z-axis) of the schamatic.
<a name="schematicOffset"></a>Offset | [`integer`] | Specifies the relative offset of the schematic. This value is relative to the most minimal point in the schematics area. The default value if not provided is `[0, 0, 0]`. This may be used when incorperating the area of blocks defined by this schematic to a larger area to place the blocks relative to a selected point.
<a name="schematicPaletteLength"></a>PaletteLength | `integer` | **Required.** Specifies the size of the block palette.
<a name="schematicPalette"></a>Palette | [Palette Object](#paletteObject) | **Required.** Specifies the block palette. This is a mapping of block states to indices which are local to this schematic. These indices are used to reference the block states from within the [BlockData array](#schematicBlockData). It is recommeneded for maximum data compression that your indices start at zero and skip no values. The maximum index cannot be greater than [`PaletteLength - 1`](#schematicPaletteLength).
<a name="schematicBlockData"></a>BlockData | `byte[]` | **Required.** Specifies the main storage array which contains `Width * Height * Length` entries packed tightly within the array. Each entry refers to an index within the [Palette](#schematicPalette) and is `ceil(lg(PaletteLength-1))` bits in length (lg is the base 2 logarithm). The total data is padded to a multiple of 8 to fit within the byte array. The entries are indexed by `x + y * Width + z * Width * Height`.
<a name="schematicTileEntities"></a>TileEntities | [[TileEntity Object](#tileEntityObject)] | Specifies additional data for blocks which require extra data. If no additional data is provided for a block which nornally requires extra data then it is assumed that the TileEntity for the block is initialized to its default state.
<a name="schematicEntities"></a>Entities | [[Entity Object](#entityObject)] | Specifies any entities which exist within the area of this schematic.

#### <a name="metadataObject"></a> Metadata Object

An object which provides optional additional meta information about the schematic. The fields outlined here are guidelines to assist with standardization but it is recommended that any program reading and writing schematics persist all fields found within this object.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="metadataName"></a>Name | `string` | The name of the schematic.
<a name="metadataAuthor"></a>Author | `string` | The name of the author of the schematic.
<a name="metadataDate"></a>Date | `string` | The date that this schematic was created on.
<a name="metadataRequiredMods"></a>RequiredMods | [`string`] | An array of mod ids which have blocks which are referenced by this schematic's defined [Palette](#schematicPalette).

##### Metadata Object Example:

```js
{
    "Name": "My Schematic",
    "Author": "Author Name",
    "RequiredMods": [
        "a_mod",
        "another_mod"
    ]
}
```

#### <a name="paletteObject"></a>Palette Object

An object which holds a mapping of a block state id to an index. The indices are recommended to start at `0` and may be no more than [`PaletteLength - 1`](#schematicPaletteLength).

#### Block State Ids

The format of the Block State identifier is the id of the block type and a set of property `key=value` pairs surrounded by square brackets. If the block has no properties then they can be excluded. For example the air block has no properties so its id representation would be just the block type id `minecraft:air`. The planks block however has an enum property for the `variant` so its id would be `minecraft:planks[variant=oak]`.


##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="paletteEntry"></a>{blockstate} | `integer` | A single entry mapping a blockstate to an index.

##### Palette Object Example

```js
"Palette" {
    "minecraft:air": 0,
    "minecraft:planks[variant=oak]": 1,
    "a_mod:custom": 2
}
```

#### <a name="tileEntityObject"></a>Tile Entity Object

An object to specify a tile entity which is within the region. Tile entities are used by Minecraft to store additional data for a block (such as the lines of text on a sign). The fields used to describe a tile entity vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="tileEntityVersion"></a>ContentVersion | `integer` | **Required.** A version identifier for the contents of this tile entity. Used for providing better backwords compatibility.
<a name="tileEntityPos"></a>Pos | [`integer`] | **Required.** The position of the tile entity relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffsetX) applied). Must contain exactly 3 integer values.
<a name="tileEntityId"></a>Id | `string` | **Required.** The id of the tile entity type defined by this Tile Entity Object. This should be used to identify which fields should be required for the definition of this type.

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

#### <a name="entityObject"></a> Entity Object

An object to specify an entity which is within the region. The fields used to describe a tile entity vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="entityVersion"></a>ContentVersion | `integer` | **Required.** A version identifier for the contents of this tile entity. Used for providing better backwords compatibility.
<a name="entityPos"></a>Pos | [`double`] | **Required.** The position of the tile entity relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffsetX) applied). Must contain exactly 3 `double` values.
<a name="entityId"></a>Id | `string` | **Required.** The id of the entity type defined by this Entity Object. This should be used to identify which fields should be required for the definition of this type.

##### Entity Object Example

An example of possible storage of a falling sand entity. See the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Entity_format) for a complete listing of data used to store various types of entities present in vanilla minecraft. Mods may store additional data or have additional types of entities.

```js
{
    "ContentVersion": 0,
    "Pos": [0.3, 1.6, 0.3],
    "Id": "minecraft:FallingSand",
    "Block": "minecraft:sand",
    "Time": 43
}
```
