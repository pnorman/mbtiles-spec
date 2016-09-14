# MBTiles 2.0

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

# Sub-sections:

* **Interaction**: HTTP endpoints needed for implementing interactivity
* **UTFGrid**: This specification relies on [UTFGrid 1.2](https://github.com/mapbox/utfgrid-spec) for interactivity.

## Abstract

MBTiles is a specification for storing tiled map data in
[SQLite](http://sqlite.org/) databases for immediate usage and for transfer.
MBTiles files, known as **tilesets**, MUST implement the specification below
to ensure compatibility with devices.

## Database Specifications

Tilesets SHALL be valid SQLite databases of
[version 3.0.0](http://sqlite.org/formatchng.html) or higher.
Only core SQLite features are permitted; tilesets SHALL NOT require extensions.

MBTiles tilesets MAY use [the officially assigned magic number](http://www.sqlite.org/src/artifact?ci=trunk&filename=magic.txt)
to be easily identified as MBTiles.

## Database

Note: the schemas outlined are meant to be followed as interfaces.
SQLite views that produce compatible results MAY be used instead.
For convenience, this specification refers to tables and virtual
tables (views) as tables.

### Metadata

#### Schema

The database MUST contain a table or view named `metadata`.

This table MUST yield exactly two columns named `name` and
`value`. A typical create statement for the `metadata` table:

    CREATE TABLE metadata (name text, value text);

#### Content

The metadata table is used as a key/value store for settings.

The following keys are REQUIRED

* `name`: The plain-english name of the tileset.
* `type`: `overlay` or `baselayer`
* `version`: The version of the tileset, as a plain number.
* `description`: A description of the layer as plain text.

The following keys are RECOMMENDED

* `format`: The MIME format of the tile data.
* `compression`: The type of compression applied on top of the tile data. The value SHALL a valid [HTTP Content](http://www.iana.org/assignments/http-parameters/http-parameters.xhtml#content-coding) name.

  Decoders MAY assume per-format defaults if the key is not present.

The following keys are OPTIONAL

* `bounds`: The maximum extent of the rendered map area. Bounds must define an
  area covered by all zoom levels. The bounds are represented in `WGS:84` -
  latitude and longitude values, in the OpenLayers Bounds format -
  **left, bottom, right, top**. Example of the full earth: `-180.0,-85,180,85`.
* `attribution`: An attribution string, which explains in English (and HTML) the sources of
  data and/or style for the map.

Additional keys MAY be added, such as those in [UTFGrid-based interaction](https://github.com/mapbox/utfgrid-spec).

### Tiles

#### Schema

The database MUST contain a table named `tiles`.

The table MUST yield four columns named `zoom_level`, `tile_column`,
`tile_row`, and `tile_data`. A typical create statement for the `tiles` table:

    CREATE TABLE tiles (zoom_level integer, tile_column integer, tile_row integer, tile_data blob);

#### Content

The tiles table contains tiles and the values used to locate them.
The `zoom_level`, `tile_column`, and `tile_row` columns MUST the
[Tile Map Service Specification](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification) in
their construction, except the [global-mercator](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification#global-mercator) (aka Spherical Mercator) MUST be used.

The `tile_data blob` column contains raw tile data in binary. If the `format` or `compression` metadata keys are present the tile data MUST be in that format and/or compression. Regardless of the presence of those metadata keys, all tile data MUST be in the same format and compression.

### Grids

_See the [UTFGrid specification](https://github.com/mapbox/utfgrid-spec) for
implementation details of grids and interaction metadata itself: the MBTiles
specification is only concerned with storage._

#### Schema

The database MAY have optional tables named `grids` and `grid_data`.

The `grids` table MUST yield four columns named `zoom_level`, `tile_column`,
`tile_row`, and `grid`. A typical create statement for the `grids` table:

    CREATE TABLE grids (zoom_level integer, tile_column integer, tile_row integer, grid blob);

The `grid_data` table MUST yield five columns named `zoom_level`, `tile_column`,
`tile_row`, `key_name`, and `key_json`. A typical create statement for the `grid_data` table:

    CREATE TABLE grid_data (zoom_level integer, tile_column integer, tile_row integer, key_name text, key_json text);

#### Content

The `grids` table MUST contain UTFGrid data, gzip compressed.

The `grid_data` table MUST contain grid key to value mappings, with values encoded
as JSON objects.
