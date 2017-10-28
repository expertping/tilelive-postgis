# tilelive-postgis

[![npm version](https://img.shields.io/npm/v/tilelive-postgis.svg)](https://www.npmjs.com/package/tilelive-postgis)
[![npm downloads](https://img.shields.io/npm/dt/tilelive-postgis.svg)](https://www.npmjs.com/package/tilelive-postgis)
[![Build Status](https://travis-ci.org/stepankuzmin/tilelive-postgis.svg?branch=master)](https://travis-ci.org/stepankuzmin/tilelive-postgis)

Implements the tilelive API for generating mapnik vector tiles from PostGIS.

## Installation

```shell
npm install tilelive-postgis
```

## Usage

```js
const tilelive = require('@mapbox/tilelive');
require('tilelive-postgis').registerProtocols(tilelive);

const uri = 'postgis://user:password@localhost:5432/test?table=test_table&geometry_field=geometry&srid=4326';
tilelive.load(uri, (error, source) => {
  if (error) throw error;

  source.getTile(0, 0, 0, (error, tile, headers) => {
    // `error` is an erroror object when generation failed, otherwise null.
    // `tile` contains the compressed image file as a Buffer
    // `headers` is a hash with HTTP headers for the image.
  });
});
```

If PostgreSQL server running locally and accepting connections on Unix domain socket, you can connect like this:

```js
const uri = 'postgis:///var/run/postgresql/test?table=test&geometry_field=geom';
tilelive.load(uri, (error, source) => { ... });
```

You can also use `query` parameter to specify sub-query like this:

```js
const query = `(select * from schemaName.tableName where st_intersects(geometry, !bbox!)) as query`;
const uri = `postgis://user@localhost/test?table=test_table&geometry_field=geometry&query=${encodeURI(query)}`;
tilelive.load(uri, (error, source) => { ... });
```

## Parameters

The only parameter that is not mapnik specific is:

| *parameter*       | *value*  | *description* | *default* |
|:------------------|----------|---------------|----------:|
| layerName         | string   | name of the layer in resulting tiles. | defaults to the table name for backwards compatibility.|


Actual list of parameters you can see [here](https://github.com/mapnik/mapnik/wiki/PostGIS).

| *parameter*       | *value*  | *description* | *default* |
|:------------------|----------|---------------|----------:|
| table                 | string       | name of the table to fetch. | |
| geometry_field        | string       | name of the geometry field, in case you have more than one in a single table. This field and the SRID will be deduced from the query in most cases, but may need to be manually specified in some cases.| |
| geometry_table        | string       | name of the table containing the returned geometry; for determining RIDs with subselects | |
| srid                  | integer      | srid of the table, if this is > 0 then fetching data will avoid an extra database query for knowing the srid of the table | 0 |
| extent                | string       | maxextent of the geometries | determined by querying the metadata for the table |
| extent_from_subquery  | boolean      | evaluate the extent of the subquery, this might be a performance issue | false |
| connect_timeout       | integer      | timeout is seconds for the connection to take place | 4 |
| persist_connection    | boolean      | choose whether to share the same connection for subsequent queries | true |
| row_limit             | integer      | max number of rows to return when querying data, 0 means no limit | 0 |
| cursor_size           | integer      | if this is > 0 then server cursor will be used, and will prefetch this number of features | 0 |
| initial_size          | integer      | initial size of the stateless connection pool | 1 |
| max_size              | integer      | max size of the stateless connection pool | 10 |
| multiple_geometries   | boolean      | whether to use multiple different objects or a single one when dealing with multi-objects (this is mainly related to how the label are used in the map, one label for a multi-polygon or one label for each polygon of a multi-polygon)| false |
| encoding              | string       | internal file encoding | utf-8 |
| simplify_geometries   | boolean      | whether to automatically [reduce input vertices](http://blog.cartodb.com/post/20163722809/speeding-up-tiles-rendering). Only effective when output projection matches (or is similar to) input projection. Available from version 2.1.x up. | false |
| max_async_connection  | integer       | max number of PostGIS queries for rendering one map in asynchronous mode. Full doc [here](Postgis-async). Default value (1) has no effect. | 1 |
