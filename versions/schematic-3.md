 # Sponge Schematic Specification

#### Version 3

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Introduction

This specification defines a format which describes a region of a [Minecraft](http://minecraft.net) world for the purpose of serialization and storage of the region to disk. It is designed in order to allow maximum cross-compatibility between platforms, versions, and various states of modification.

## Revision History

Version | Date | Notes
--- | --- | ---
1 | 2016-08-23 | Initial Version
2 | 2019-05-08 | Entity and Biome support and DataVersion additions
3 | 2021-05-04 | 3D Biome support <br> Rename Palette to BlockPalette <br> Wordsmithing varint and palette usages


## Definitions

##### <a name="defNBT"></a>NBT
[Named Binary Tag](http://minecraft.gamepedia.com/NBT_format) (NBT) is a binary format used by Minecraft for various files and storage of data. Its many supported types are used throughout this specification as a means of what *expected* tags are used, and how the data is stored using the tag type.

##### <a name="defResourceLocation"></a>Resource Location
Resources identified by a Resource Location have an identifier string which is made up of two sections, the domain and the resource name, written as `domain:name`. If the domain is not specified then it is assumed to be `minecraft`. For example `planks` would identify the same resource as `minecraft:planks` as the `minecraft:` prefix is implicit. Many examples of resource locations can be found [on the Minecraft wiki for Java Edition](https://minecraft.gamepedia.com/Java_Edition_data_values).

##### <a name="defBlockState"></a>Block State
A Block State is an instance of a block type with a set of extra data used to further define the material or properties. The additional data varies by block type and a complete listing of vanilla types is available [here](http://minecraft.gamepedia.com/Block_states). Mods can add additional block types and properties. A state is always referencable by a [resource location](#defResourceLocation).

##### <a name="defBlockEntity"></a>BlockEntity
A special Entity that resides at a position with a correlated [blockstate](#defBlockState). Specifically, these are instances that can have complex data contained within them and may perform various operations in the world and as a result, their data is stored separately and otherwise too complex to store as a [`BlockState`](#defBlockState). Examples of these include chests, furnaces, beacons, etc. where their data is specific to each position they may exist. More information on these can be read on the [Minecraft wiki](https://minecraft.gamepedia.com/Block_entity).

##### <a name="defEntityType"></a>Entity Type
An Entity Type is a literal type of an `Entity` that is represented as a [Resource Location](#defResourceLocation). The additional data varies by entity type and a complete listing of vanilla types is available [here](https://minecraft.gamepedia.com/Java_Edition_data_values#Entity_IDs).

##### <a name="defEntity"></a>Entity
An instance of an [entity type](#defEntityType) that has a relative position to the origin of the schematic. Additional data is dependent on the type while vanilla optional data for entities can be found [here](https://minecraft.gamepedia.com/Chunk_format#Entity_format). Mods can add extra data to any entity instance. Each entity in a Minecraft game instance is uniquely identifable through a [`UUID`](https://docs.oracle.com/javase/6/docs/api/java/util/UUID.html), however schematics **can not** be relied on to provide a `UUID` for the targeted entity as common Minecraft implementations will generate a new `UUID` for each entity spawned from a schematic.

##### <a name="defBiome"></a>Biome
A specific environmental aspect to a region that affects various rendering options, such as foliage coloring, rain, snow, temperature, mob spawning, and sometimes sky rendering. They do have potential physical effects based on implementation and are id'ed by [`ResourceLocation`s](#defResourceLocation). More about these can be read [on the Minecraft wiki](https://minecraft.gamepedia.com/Biome).

## Data Types

A short explanation of the various data types used in schematics. A conversion is provided for each data type as [NBT](#defNBT) is used for storage. To find more information on how NBT tags are stored in binary format, click [here](http://web.archive.org/web/20110723210920/http://www.minecraft.net/docs/NBT.txt).

Specification Data Type | NBT Equivalent | Format Stored to target NBT Equivalent
---|:---:|---
`integer` | `NBT Int` | No conversion
`short` | `NBT Short` | No conversion
`byte` | `NBT Byte` | No conversion
`long` | `NBT Long` | No conversion
`float` | `NBT Float` | No conversion
`double` | `NBT Double` | No conversion
`unsigned short` | `NBT Short` | Uses the 16 bits of the signed `NBT Short` to store an unsigned short. The value can be extracted to an integer in Java by using `value & 0xFFFF`.
`integer[]` | `NBT Int Array` | No conversion
`varint[]` | `NBT Byte Array` | Each `integer` is bitpacked with [varint](https://wiki.vg/VarInt_And_VarLong) encoding. A Examples can be found with Sponge's implementation for [retreving data](https://github.com/SpongePowered/SpongeCommon/blob/aa2c8c53b4f9f40297e6a4ee281bee4f4ce7707b/src/main/java/org/spongepowered/common/data/persistence/SchematicTranslator.java#L147-L175) and [storing data](https://github.com/SpongePowered/SpongeCommon/blob/aa2c8c53b4f9f40297e6a4ee281bee4f4ce7707b/src/main/java/org/spongepowered/common/data/persistence/SchematicTranslator.java#L230-L251).
`float[]` | `NBT List of NBT Float` | Each `float` is stored as an `NBT Float` in an indexed `NBT List`
`double[]` | `NBT List of NBT Double` | Each `double` is stored as an `NBT Double` in an indexed `NBT List`
`string` | `NBT String` | No conversion
`string[]` | `NBT List of NBT String` | No conversion
`Object` | `NBT Compound` | No conversion, each object should serialize to supported types within a compound
`Object[]` | `NBT List of NBT Compound` | No conversion, each `Object` is stored in an `NBT List` of `NBT Compound`s as indexed according to the `Object[]`.
`extra` | `NBT Compound` | No conversion. The content of the compound is specified by Minecraft, not this format.


## Specification

### Format

The structure described by this specification is persisted to disk using the [Named Binary Tag](http://minecraft.gamepedia.com/NBT_format) (NBT) format. Before writing to disk the NBT data must be compressed using the [GZip](https://www.gnu.org/software/gzip/) data compression algorithm. The highly recommended file extension for files using this specification is `.schem` (chosen so as to not conflict with the legacy `.schematic` format allowing easy distinction between the two).

All field names in the specification are **case sensitive**.

The [Schematic](#schematicObject) schema compound is nested inside the root compound, like with most of Minecraft's own data files:
```json
{
    "": {
        "Schematic": {
            //...
        }
    }
}
```

### Schema

#### <a name="schematicObject"></a>Schematic Object

This is the root object for the specification.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="schematicVersion"></a>Version | `integer` | **Required.** Specifies the format version being used. It may be used to provide validation and auto-conversion from older versions. The current version is `3`.
<a name="schematicDataVersion"></a>DataVersion | `integer` | **Required.** Specifies the data version of Minecraft that was used to create the schematic. This is to allow for block and entity data to be validated and auto-converted from older versions. This is dependent on the Minecraft version, eg. Minecraft 1.12.2's data version is [1343](https://minecraft.gamepedia.com/1.12.2).
<a name="schematicMetadata"></a>Metadata | [Metadata Object](#metadataObject) | Provides optional metadata about the schematic.
<a name="schematicWidth"></a>Width | `unsigned short` | **Required.** Specifies the width (the size of the area in the X-axis) of the schematic.
<a name="schematicHeight"></a>Height | `unsigned short` | **Required.** Specifies the height (the size of the area in the Y-axis) of the schematic.
<a name="schematicLength"></a>Length | `unsigned short` | **Required.** Specifies the length (the size of the area in the Z-axis) of the schematic.
<a name="schematicOffset"></a>Offset | `integer[3]` | Specifies the relative offset of the schematic from the paster. When pasting, if there is a reasonable location to use as a base position, implementations SHOULD offset the location of the paste by this vector. The default value if not provided is `[0, 0, 0]`. Example: If a player is pasting from `1, 2, 3`, and the offset is `4, 5, 6`, then the first block should be placed at `5, 7, 9`
<a name="schematicLength"></a>Blocks | [Block Container](#blockContainer) | Specifies Block related data such as placing `BlockState`s and `BlockEntity` instances.
<a name="schematicLength"></a>Biomes | [Biome Container](#biomeContainer) | Specifies Biome related data.
<a name="schematicEntities"></a>Entities | [Entity Object](#entityObject)[] | Specifies entities to be placed in the schematic. If no additional data is provided for an [entity type](#defEntityType) which normally requires extra data, then it is assumed that the Entity is initialized with all defaults.
#### <a name="metadataObject"></a> Metadata Object

An object which provides optional additional meta information about the schematic. The fields outlined here are guidelines to assist with standardization but it is recommended that any program reading and writing schematics persist all fields found within this object.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="metadataName"></a>Name | `string` | The name of the schematic.
<a name="metadataAuthor"></a>Author | `string` | The name of the author of the schematic.
<a name="metadataDate"></a>Date | `long` | The date that this schematic was created on. This is specified as milliseconds since the Unix epoch.
<a name="metadataRequiredMods"></a>RequiredMods | `string[]` | An array of mod ids which have blocks which are referenced by this schematic's defined [Palette](#schematicPalette).

##### Metadata Object Example:

```json
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

An object which holds a mapping of a resourced id to an index.

##### Palette Object Example

```json
{
    "minecraft:air": 0,
    "minecraft:planks[variant=oak]": 1,
    "a_mod:custom": 2
}
```

##### BlockState Palette Specifics

The format of the BlockState identifier is the id of the block type and a set of comma-separated property `key=value` 
pairs surrounded by square brackets. If the block has no properties then the square brackets can be omitted. The block
type id is specified as a [Resource Location](#defResourceLocation), for a full list of available block IDs, refer to 
the particular Minecraft Java edition version of the official wiki.

For example the air block has no properties so its id representation would be just the block type id `minecraft:air`. 
The wheat block has an integer property for the `age` so its id would be `minecraft:wheat[age=3]`. 
Properties ordering is nondeterministic, unknown properties for the particular game version may result in errors.

##### Biome Palette Specifics

Biomes are simply referenced by their game's particular [Resource Location](#defResourceLocation). Due to the nature of
game specific registries, it's necessary to consider that a biome may not exist between different game versions or
game setups, such that a biome may only exist as part of a [DataPack](https://minecraft.fandom.com/wiki/Data_Pack), or 
mod. A full list of available biomes for the vanilla game are available [here](https://minecraft.fandom.com/wiki/Biome#Overworld).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="blockPaletteEntry"></a>{blockstate} | `integer` | A single entry mapping a blockstate to an index.
<a name="biomePaletteEntry"></a>{biome} | `integer` | A single entry mapping a biome to an index


#### <a name="blockContainer"></a> Block Container 

An object which provides block based data and has particular requirements alone. Not all schematics need to consist of blocks.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="schematicPalette"></a>Palette | [Palette Object](#paletteObject) |  **Required** Specifies the palette. This is a mapping of block states to indices which are local to this schematic. These indices are used to reference the block states from within the [Data](#schematicBlockData) array. It is recommended for maximum data compression that your indices start at zero and skip no values.
<a name="schematicBlockData"></a>Data | `varint[]` | **Required** Specifies the main storage array which contains `Width * Height * Length` entries. Each entry is specified as a varint and refers to an index within the [Palette](#schematicPalette). The entries are indexed by `x + z * Width + y * Width * Length`.
<a name="schematicBlockEntities"></a>BlockEntities | [BlockEntity Object](#blockEntityObject)[] | Specifies additional data for blocks which require extra data. If no additional data is provided for a block which normally requires extra data then it is assumed that the BlockEntity for the block is initialized to its default state.

#### <a name="biomeContainer"></a> Biome Container 

An object which provides biome based data and has particular requirements alone. Not all schematics need to consist of biomes.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="schematicBiomePalette"></a>Palette | [Palette Object](#paletteObject) | **Required** Specifies the palette. This is a mapping of [biomes](#defBiome) to indices which are local to this schematic. These indices are used to reference the biomes from within the [Data](#schematicBiomeData) array. It is recommended for maximum data compression that your indices start at zero and skip no values.
<a name="schematicBiomeData"></a>Data | `varint[]` | **Required** Specifies the main storage array which contains `Width * Height * Length` entries for `Biome`s at positions. Each entry is specified as a varint and refers to an index within the [Palette](#schematicBiomePalette). The entries are indexed by `x + z * Width + y * Width * Length`.


#### <a name="blockEntityObject"></a>BlockEntity Object

An object to specify a `BlockEntity` which is within the region. Block entities are used by Minecraft to store additional data for a block (such as the lines of text on a sign). The fields used to describe a `BlockEntity` vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="blockEntityPos"></a>Pos | `integer[3]` | **Required.** The position of the `BlockEntity` relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffset) applied). Must contain exactly 3 integer values.
<a name="blockEntityId"></a>Id | `string` | **Required.** The id of the `BlockEntity` type defined by this `BlockEntity` Object, specified as a [Resource Location](#defResourceLocation). This should be used to identify which fields should be required for the definition of this type.
<a name="blockEntityData"></a>Data | `extra` | **Optional** The extra information related to a `BlockEntity` that otherwise would define various bits of information for the `BlockEntity` associated by their respective types.

##### BlockEntity Object Example

An example of possible storage of a sign. See the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format) for a complete listing of data used to store various types of block entities present in vanilla minecraft. Mods may store additional data or have additional types of block entities.

```json
{
    "Pos": [0, 1, 0],
    "Id": "minecraft:sign",
    "Data": {
        "Text1": "foo",
        "Text2": "",
        "Text3": "bar",
        "Text4": ""
    }
}
```

#### <a name="entityObject"></a>Entity Object
An object to specify an instance of an [entity type](#defEntityType) within the region. Entities usually occupy a three dimensional space within the region and have many characteristics, varied by the entity type, however the structure will be the same as used by the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format), dependent on the `Data Version` the schematic is created from.

##### Fields
Field Pattern | Type | Description
---|:---:|---
<a name="entityPos"></a>Pos | `double[3]` | **Required.** The position of the entity relative to the `[0, 0, 0]` position of the schematic (without the offset applied). Must contain exactly 3 double values.
<a name="entityId"></a>Id | `string` | **Required.** The id of the entity type defined by this `Entity` Object, specified as a [Resource Location](#defResouceLocation). This should be used to identify which fields should be required for the definition of this type.
<a name="entityData"></a>Data | `extra` | **Optional** The extra information related to an `Entity` that otherwise is either defaulted or required by the entity associated with the [entity type](#defEntityType)

##### Entity Object Example

An example of possible storage of a creeper. See the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format) for a complete listing of data used to store various types of entities present in vanilla Minecraft. Mods may store additional data or have additional types of entities.

```json
{
    "Pos": [15.0043293, 68.000321, 40.452],
    "Id": "minecraft:creeper",
    "Data": {
        "Motion": [0.00203, 0.00203, 1.000],
        "Rotation": [189.30, 45],
        "Fire": -20,
        "NoGravity": 0,
        "CustomName": "Sheep",
        "CustomNameVisible": 1
    }
}
```
