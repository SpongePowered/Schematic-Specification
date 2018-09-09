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
2 | 2018-08-19 | More features, easier handling, see [Changelog](#changelog)

## Specification

### Format

All field names in the specification are **case sensitive**.

#### <a name="formFile"></a>File

The structure described by this specification is persisted to disk using the [Named Binary Tag](http://minecraft.gamepedia.com/NBT_format) (NBT) format (also see [here](https://wiki.vg/NBT)). Before writing to disk the NBT data must be compressed using the [GZip](https://www.gnu.org/software/gzip/) data compression algorithm. The highly recommended file extension for files using this specification is `.schem` or `.spongeschem` (chosen so as to not conflict with the legacy `.schematic` format allowing easy distinction between the two).

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
<a name="schematicMetadata"></a>Metadata | [Metadata Object](#metadataObject) as `NBT compound` | **Optional.** Provides metadata about the schematic. This shall be saved as [NBT](#formFile) tag compound.
<a name="schematicWidth"></a>Width | `NBT int` | **Required.** Specifies the width (the amount of blocks along the X-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicHeight"></a>Height | `NBT int` | **Required.** Specifies the height (the amount of blocks along the Y-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicLength"></a>Length | `NBT int` | **Required.** Specifies the length (the amount of blocks along the Z-axis) of the schematic, must be interpreted as unsigned. This shall be saved as [NBT](#formFile) tag int.
<a name="schematicOffset"></a>Offset | `NBT int array`[3] | **Optional.** Specifies the relative offset of the schematic. This value is relative to the most minimal point in the schematics area. If present, this offset shall be added to all blocks, tiles and entities when pasting it into the world (may be negative). This shall be saved as [NBT](#formFile) tag int array.
<a name="schematicPaletteMax"></a>PaletteMax | `NBT byte` | **Required.** Specifies the size of the block palette index in number of bytes needed for the maximum palette index. Implementations must use this to determine, which index format will be needed. The palette index may fit within a datatype smaller than a 32-bit integer or even a longer datatype like 64-bit integer. This shall be saved as [NBT](#formFile) tag byte.
<a name="schematicPalette"></a>Palette | `NBT list` of [Palette Object](#paletteObject) as `NBT compound` | **Required.** Specifies the block palette. This is a mapping of block states to indices which are local to this schematic. The array element indices are used to reference the block states from within the [BlockData array](#schematicBlockData). The maximum index cannot be greater than [`2^(PaletteMax * 8) - 1`](#schematicPaletteMax).  This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag string.
<a name="schematicBlockData"></a>BlockData | `nbt int array` or `nbt long array` | **Required.** Specifies the main storage array which contains `Width * Height * Length` entries. Each entry is specified as a `int` or `long`, depending on the given [PaletteMax](#schematicPaletteMax) (1 to 4 bytes = int; 5 to 8 bytes = long). This index refers to the array element index within the [Palette](#schematicPalette). The entries are indexed by `x + z * Width + y * Width * Length`. This shall be saved as [NBT](#formFile) tag int array or [NBT](#formFile) tag long array.
<a name="schematicTileEntities"></a>TileEntities | `NBT list` of [TileEntity Object](#tileEntityObject) as `NBT compound` | **Optional.** Specifies additional data for blocks which require extra data. If no additional data is provided for a block which normally requires extra data then it is assumed that the TileEntity for the block is initialized to its default state. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag compound.
<a name="schematicEntities"></a>Entities | `NBT list` of [Entity Object](#entityObject) as `NBT compound` | **Optional.** Specifies all entities, within the area of this schematic. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag compound.


#### <a name="metadataObject"></a>Metadata Object

An object which provides optional additional meta information about the schematic. The fields outlined here are guidelines to assist with standardization but it is recommended that any program reading and writing schematics persist all fields found within this object.

##### Fields

Field Name | Type | Description
---|:---:|---
<a name="metadataName"></a>Name | `NBT string` | **Optional.** The name of the schematic. This shall be saved as [NBT](#formFile) tag string.
<a name="metadataAuthor"></a>Author | `NBT string` | **Optional.** The name of the author of the schematic. This shall be saved as [NBT](#formFile) tag string.
<a name="metadataDate"></a>Date | `NBT long` | **Optional.** The date that this schematic was created on. This is specified as seconds since the Unix epoch. This shall be saved as [NBT](#formFile) tag long.
<a name="metadataRequiredMods"></a>RequiredMods | `NBT list` of `NBT string` | **Optional.** An array of mod ids which have blocks which are referenced by this schematic's defined [Palette](#schematicPalette) or other contents of the schematic (e.g. [Tiles](#tileEntityObject) or [Entities](#entityObject)). May be empty. This shall be saved as [NBT](#formFile) tag list of NBT tag string.
<a name="metadataOptionalMods"></a>OptionalMods | `NBT list` of `NBT string` | **Optional.** An array of mod ids which are needed to fully provide all features saved within this schematic. May be empty. This shall be saved as [NBT](#formFile) tag list of NBT tag string.

##### Metadata Object Example:

```NBT
{
    Name: "My Schematic",
    Author: "Author Name",
    Date: 1534701119L,
    RequiredMods: [
        "a_mod",
        "another_mod"
    ],
    OptionalMods: [
        "CraftBukkit"
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

```NBT
Palette: [
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
<a name="tileEntityProp"></a>{properties} | `<?>` | **Required**. Amount and format of fields depending on the tile entity, specified by Mojang. As listed within the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format).
<a name="tileEntityPosX"></a>x | `NBT int` | **Required**. As specified by chunk format above, BUT this value must to be converted to relative coordinates of the area. Relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffset) applied).
<a name="tileEntityPosY"></a>y | `NBT int` | **Required**. As specified by chunk format above, BUT this value must to be converted to relative coordinates of the area. Relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffset) applied).
<a name="tileEntityPosZ"></a>z | `NBT int` | **Required**. As specified by chunk format above, BUT this value must to be converted to relative coordinates of the area. Relative to the `[0, 0, 0]` position of the schematic (without the [offset](#schematicOffset) applied).
<a name="tileEntityAdd"></a>{additions} | `<?>` | **Optional.** May be expanded by custom values needed for modifications. The corresponding Mod should at least be listed as optional within the [Metadata object](#metadataObject).

##### Tile Entity Object Example

An example of possible storage of a sign. See the [Minecraft Chunk Format](http://minecraft.gamepedia.com/Chunk_format#Block_entity_format) for a complete listing of data used to store various types of tile entities present in vanilla minecraft. Mods may store additional data or have additional types of tile entities.

```NBT
{
    x: 0,
    y: 1,
    z: 2,
    id: "minecraft:sign",
    Text1: "foo",
    Text2: "",
    Text3: "bar",
    Text4: ""
}
```

#### <a name="entityObject"></a>Entity Object

An object to specify all entitys which are within the area. Entities are used by Minecraft for movable or interactable objects (such as mobs or minecarts). The fields used to describe a entity vary for each type, however the structure will be the same as used by the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format).

##### Fields

Field Pattern | Type | Description
---|:---:|---
<a name="entityProp"></a>{properties} | `<?>` | **Required**. Amount and format of fields depending on the entity, specified by Mojang. As listed within the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format).
<a name="entityPos"></a>Pos | `NBT list` of `NBT double`[3] | **Required.** As specified by chunk format above, BUT this value must to be converted to relative coordinates of the area. Relative to the `[0.0, 0.0, 0.0]` position of the schematic (without the [offset](#schematicOffset) applied). Must contain exactly 3 double values. This shall be saved as [NBT](#formFile) tag list of [NBT](#formFile) tag double.
<a name="entityUUIDHigh"></a>UUIDMost | `NBT long` | **Required.** As specified by chunk format above, BUT a implementation must check if the UUID is already present and if so, replace this one by another.
<a name="entityUUIDLow"></a>UUIDLeast | `NBT long` | **Required.** As specified by chunk format above, BUT a implementation must check if the UUID is already present and if so, replace this one by another.
<a name="entityAdd"></a>{additions} | `<?>` | **Optional.** May be expanded by custom values needed for modifications (e.g. CraftBukkit adds `Bukkit.updateLevel`). The corresponding Mod should at least be listed as optional within the [Metadata object](#metadataObject).

##### Entity Object Example

An example of possible storage of a sign. See the [Minecraft Chunk Format](https://minecraft.gamepedia.com/Chunk_format#Entity_format) for a complete listing of data used to store various types of tile entities present in vanilla minecraft. Mods may store additional data or have additional types of entities.

```NBT
{
    AgeLocked: 0b,
    HurtByTimestamp: 0,
    Attributes: [
        { Base: 10.0d, Name: "generic.maxHealth" },
        { Base: 0.0d, Name: "generic.knockbackResistance" },
        { Base: 0.20000000298023224d, Name: "generic.movementSpeed" },
        { Base: 0.0d, Name: "generic.armor" },
        { Base: 0.0d, Name: "generic.armorToughness" },
        { Base: 16.0d, Modifiers: [
            {
                UUIDMost: 4449189467270498881L,
                UUIDLeast: -8942318547875874041L,
                Amount: -0.05492870821106577d,
                Operation: 1,
                Name: "Random spawn bonus"
            }
        ],
        Name: "generic.followRange" }
    ],
    Invulnerable: 0b,
    FallFlying: 0b,
    ForcedAge: 0,
    PortalCooldown: 0,
    AbsorptionAmount: 0.0f,
    FallDistance: 0.0f,
    InLove: 0,
    DeathTime: 0s,
    HandDropChances: [ 0.085f, 0.085f ],
    PersistenceRequired: 1b,
    Spigot.ticksLived: 74919199,
    Age: 0,
    Motion: [ 0.0d, -0.0784000015258789d, 0.0d ],
    Leashed: 0b,
    UUIDLeast: -7755591127829794293L,
    Health: 10.0f,
    LeftHanded: 0b,
    Air: 300s,
    OnGround: 1b,
    Dimension: 0,
    Rotation: [ 194.99161f, 0.0f ],
    HandItems: [ {}, {} ],
    ArmorDropChances: [ 0.085f, 0.085f, 0.085f, 0.085f ],
    UUIDMost: 7781923390897078472L,
    Pos: [ 121.25787019796437d, 69.0d, 139.3946334861942d ],
    Fire: -1s,
    ArmorItems: [ {}, {}, {}, {} ],
    CanPickUpLoot: 0b,
    HurtTime: 0s
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
* Changed [Pos](#tileEntityPosX) of tiles to x, y, z. Because mojangs specified it like this, so there is **no more conversion necessary**.
* Added [OptionalMods](#metadataOptionalMods) to Metadata. Because there often are only optional additional **features within tiles or entities, which are not necessarily needed**.

### Other changes

* Added **OPTIONAL** or **REQUIRED** in every field to make it more clear
* Replaced the word `region` by the word `area`, to avoid misunderstanding since a minecraft world already has so called `regions`
* Made the offset more clear
* Moved Definition#BlockState to chapter PaletteObject
* Added the corresponding NBT type to every field
