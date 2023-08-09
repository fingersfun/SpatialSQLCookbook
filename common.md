# 1. Create a point geometry from a latitude/longitude pair
## PostGIS

SELECT
ST_SetSRID(ST_MakePoint(lng, lat), 4326) as point_geom
FROM table

## BigQuery

SELECT
ST_GEOGPOINT(lng, lat) as point_geom
FROM table

## Snowflake and Redshift

SELECT
ST_MakePoint(lng, lat) as point_geom
FROM table

# 2. Create a geometry from Well Known Text
## PostGIS, Redshift

SELECT
ST_GeomFromText('POINT(-71.064544 42.28787)', 4326) as geom
FROM table

## BigQuery

SELECT
ST_GEOGFROMTEXT('POINT(-71.064544 42.28787)') as geom
FROM table

## Snowflake

SELECT
ST_GEOGFROMWKT('POINT(-71.064544 42.28787)') as geom
FROM table

# 3. Change your data to a different projectio
# PostGIS and Redshift

SELECT
ST_Transform(geom, 4326) as geom
FROM table

# 4. Get a latitude and longitude from a geometry
## PostGIS, BigQuery, Snowflake, and Redshift

SELECT
ST_X(geom) as longitude,
ST_Y(geom) as latitude
FROM table

# Measurements
# 5. Find the area of a polygon
## PostGIS, BigQuery, Snowflake, and Redshift

SELECT
ST_Area(geom) as area
FROM table

# 6. Find the perimeter of a polygon
## PostGIS, BigQuery, Snowflake, and Redshift

SELECT
ST_Perimeter(geom) as area
FROM table
# 7. Find the length of a line
## PostGIS, BigQuery, Snowflake, and Redshift

SELECT
ST_Length(geom) as length
FROM table
# 8. Find the distance between to geometries
## PostGIS and BigQuery

SELECT
ST_Distance(geom_1, geom_2) as distance
FROM table
# 9. Find the shortest distance, to a specific point, between two geometries
## PostGIS and BigQuery

SELECT
ST_ClosestPoint(geom_1, geom_2) as closest_point
FROM table
# Transform geometries
# 10. Create a buffer around a geometry
## PostGIS, BigQuery, and Redshift

SELECT
ST_Buffer(geom, 100) as closest_point
FROM table
# 11. Centroid of a polygon or line
## PostGIS, BigQuery, Snowflake, and Redshift

SELECT
ST_Centroid(geom) as centroid
FROM table
# 12. Create a concave or convex hull
## PostGIS, BigQuery, and Redshift

SELECT
ST_ConvexHull(geom) as convex_hull
FROM table

SELECT
ST_ConcaveHull(geom) as convex_hull
FROM table
## You can also do this by grouping geometries:
# ST_Collect will collect the matching geometries together into a GeometryCollection

SELECT 
ST_ConvexHull(ST_Collect(geom)) as convex_hull,
category
FROM table
GROUP BY category
# 13. Union geometries together
## PostGIS and BigQuery

SELECT
ST_Union(geom) as geom,
category
FROM table
GROUP BY category
# 14. Create Voronoi polygons
## PostGIS

SELECT
ST_VoronoiPolygons(geom) as voronoi_polygons
FROM table
# 15. Find the resulting polygon of an intersection
## PostGIS, BigQuery, Redshift, and Snowflake

SELECT
ST_Intersection(geom_1, geom_2) as intersection
FROM table
## Or if you want to find the leftover parts of a geometry relations ship you can use ST_Difference:
## PostGIS, BigQuery, Redshift, and Snowflake

SELECT
ST_Difference(geom_1, geom_2) as intersection
FROM table
# Spatial Relationships
# 16. Find geometries within a certain distance
## PostGIS, BigQuery, Redshift, and Snowflake

SELECT
*
FROM table
WHERE ST_DWithin(
	geom_1, 
	ST_GeomFromText('POINT(-73.9895258 40.7413529)',4326),
	1000)
# 17. See if two geometries overlap, touch, cross, intersect, contain, etc. (or evaluate spatial relationships)

ST_Equals – returns true if the two geometries are exactly equal
ST_Intersects – returns true if the two geometries share any space in common
ST_Disjoint – returns true is the two geometries do not share any space, or the opposite of ST_intersects
ST_Crosses – Returns true if the geometries have some, but not all, interior points in common
In more detail, “returns true if the intersection results in a geometry whose dimension is one less than the maximum dimension of the two source geometries and the intersection set is interior to both source geometries”.
If the geometries intersect but also the intersection result is one less dimension (or it crosses the intersecting geometry one time) it will return true. Works for multipoint/polygon, multipoint/linestring, linestring/linestring, linestring/polygon, and linestring/multipolygon comparisons.
ST_Overlaps – returns true if the two geometries overlap spatially, or intersect but one does not completely contain the other
ST_Touches – returns true if one geometry touches another, but does not intersect the interior
ST_Within – returns true if the first geometry is completely within the second geometry
ST_Contains – returns true if the second geometry is completely contained by the first geometry (opposite of ST_Within)
## PostGIS, BigQuery, Redshift, and Snowflake

SELECT
ST_Intersects(geom_1, geom_2) as intersects
FROM table
# Other functions
# 18 . Simplify a geometry
## PostGIS

SELECT
ST_SimplifyPreserveTopology(geom, 1) as geom
FROM table

## PostGIS, BigQuery, Redshift, and Snowflake

SELECT
ST_Simplify(geom, 1) as geom
FROM table
# 19. Create random points
## PostGIS

SELECT
ST_GeneratePoints(geom, number_column) as geom
FROM table

## BigQuery

WITH point_lists AS (
  SELECT `carto-un`.carto.ST_GENERATEPOINTS(geom, number_column) AS points
  FROM table
)
SELECT points FROM point_lists CROSS JOIN point_lists.points

# 20. Cluster points
## PostGIS and BigQuery

SELECT
geom,
ST_ClusterDBSCAN(geom, desired_distance, min_geoms_per_cluster) OVER() as cluster_id
FROM table
## PostGIS and BigQuery

SELECT
geom,
ST_ClusterKMeans(geom, number_of_clusters) OVER() as cluster_id
FROM table
# spatial functions
# Single Geometry
SELECT
ST_Distance(
	geom,
	ST_GeomFromText('POINT(-73.9895258 40.7413529)')) as distance
FROM table
# Same Table
SELECT
ST_Distance(
	geom,
	(SELECT geom FROM table WHERE id = 1)) as distance
FROM table
# WHERE Clause
SELECT
*
FROM table
WHERE ST_Distance(
	geom,
	(SELECT geom FROM table WHERE id = 1)) < 100
# Joins
SELECT
a.id,
COUNT(b.id) as count
FROM table a
JOIN other_table b
ON ST_Contains(a.geom, b.geom)
GROUP BY a.id
# Cross Joins
SELECT
ST_Distance(a.geom, b.geom) as distance
a.id as id_one,
b.id as id_two
FROM table a, other_table b
# Subqueries
SELECT
a.id,
(SELECT COUNT(id) FROM table WHERE ST_DWithin(a.geom, geom, 400)) as within_400m
FROM other_table a
# Aggregations
# 21. H3 Cells
WITH
  mn AS (
  SELECT
    geom
  FROM
     project.dataset.table
  WHERE
    name = 'Minnesota')

SELECT
  h3,
  `carto-un`.carto.H3_BOUNDARY(h3) AS geom
FROM (
	SELECT `carto-un`.carto.H3_POLYFILL(geom, 5) AS cells
  FROM 
    mn),
UNNEST(cells) AS h3
# 22. Create map tiles
CALL `carto-un`.carto.CREATE_TILESET(
  R'''(
    SELECT geom, type
    FROM `carto-do-public-data.natural_earth.geography_glo_roads_410`
  )
  ''',
  R'''`cartobq.maps.natural_earth_roads`''',
  STRUCT(
    "Tileset name" AS name,
    "Tileset description" AS description,
    NULL AS legend,
    0 AS zoom_min,
    10 AS zoom_max,
    "geom" AS geom_column_name,
    NULL AS zoom_min_column,
    NULL AS zoom_max_column,
    1024 AS max_tile_size_kb,
    "RAND() DESC" AS tile_feature_order,
    true AS drop_duplicates,
    R'''
      "custom_metadata": {
        "version": "1.0.0",
        "layer": "layer1"
      }
    ''' AS extra_metadata
  )
);
# 23. Routing with roads or other networks
SELECT * FROM pgr_dijkstra(
    '
      SELECT gid AS id,
        source,
        target,
        length AS cost
      FROM ways
    ',
    1,
    2,
    directed := false);
  -- You can do the sample with the CARTO Analytics Toolbox. After creating a network, you can calculate the shortest paths:
    CALL `carto-un`.carto.FIND_SHORTEST_PATH_FROM_NETWORK_TABLE(
  "mydataset.network_table",
  "mydataset.shortest_path_table",
  "ST_GEOGPOINT(-74.0, 40.0)",
  "ST_GEOGPOINT(-75.0, 41.0)"
);
# 24. Simple nearest neighbor
## PostGIS

SELECT id, (SELECT geom FROM another_table LIMIT 1) <-> geom AS distance
FROM table
ORDER BY distance
LIMIT 1
## BigQuery

SELECT
  a.id,
  ARRAY_AGG(b.geoid ORDER BY st_distance(a.geom,
      b.geom) ASC LIMIT 1)[SAFE_OFFSET(0)] as id,
  ARRAY_AGG(st_distance(a.geom,
      b.geom) ORDER BY st_distance(a.geom,
      b.geom) ASC LIMIT 1)[SAFE_OFFSET(0)] as dist

FROM
  project.dataset.table_1 a
CROSS JOIN project.dataset.table_2 b
GROUP BY a.id
# 25. Calculate the percentage overlap between polygons
## PostGIS

WITH a AS (SELECT geom FROM table WHERE id = 1)

SELECT 
ST_Area(ST_Intersection(geom, (SELECT geom FROM a))/ST_Area(geom) AS overlap
FROM table
ORDER BY overlap
## PostGIS

SELECT
COUNT(a.column),
b.id,
b.geom
FROM table b
LEFT JOIN table_2 a 
ON ST_Intersects(a.geom, b.geom)
WHERE ST_Area(ST_Intersection(a.geom, b.geom)/ST_Area(geom) > .5
# 26. Find the distance between the two nearest points on two polygons that do not touch
## PostGIS

SELECT
  ST_Distance(ST_ClosestPoint(a.geom,
      b.geom),
    ST_ClosestPoint(b.geom,
      a.geom)) AS distance,
	ST_MakeLine(ST_ClosestPoint(a.geom,
      b.geom),
    ST_ClosestPoint(b.geom,
      a.geom)) AS geom,
  a.objectid
FROM
  table a
CROSS JOIN LATERAL (
  SELECT
    geom
  FROM
    table_2
  WHERE
    ST_Disjoint(a.geom,
      geom)
	AND a.objectid != objectid
  ORDER BY
    ST_Distance(geom,
      a.geom) ASC LIMIT 1) b
### BigQuery

SELECT
  ST_MakeLine(ST_ClosestPoint(a.geom,
      b.geom),
    ST_ClosestPoint(b.geom,
      a.geom)) AS geom,
  ST_Distance(ST_ClosestPoint(a.geom,
      b.geom),
    ST_ClosestPoint(b.geom,
      a.geom)) AS geom,
  a.id
FROM
  table_one a
LEFT JOIN (
  SELECT
    one.geom,
    two.id,
    ROW_NUMBER() OVER (PARTITION BY two.id ORDER BY ST_Distance(one.geom, two.geom) ASC) AS rank
  FROM
    table_one one,
    table_two two
  WHERE
    ST_Disjoint(one.geom,
      two.geom)
  ORDER BY
    ST_Distance(one.geom,
      two.geom) ASC ) b
USING
  (id)
WHERE
  b.rank = 1
# 27. Create a buffer around a geometry and clip out areas from the buffer
## PostGIS

SELECT 
ST_Difference(
   ST_Union(
       ST_Buffer(a.geom, 1609)
     ), 
  ST_Union(b.geom))
  as clipped_geom
FROM table a
LEFT JOIN table_2 b
ON ST_Intersects(a.geom, b.geom)

## BigQuery

SELECT 
ST_Difference(
   ST_Union_Agg(
       ST_Buffer(a.geom, 1609)
     ), 
  ST_Union(b.geom))
  as clipped_geom
FROM table a
LEFT JOIN table_2 b
ON ST_Intersects(a.geom, b.geom)

# 28. Bulk nearest neighbor joins
## PostGIS

SELECT
  a.geom,
  b.name,
  ST_Distance(a.geom, b.geom) AS distance
FROM
  table a
CROSS JOIN LATERAL
  (SELECT name, geom
   FROM table_2 b
   ORDER BY
     a.geom <--> geom
   LIMIT 1) b
## BigQuery

SELECT
  a.id,
  ARRAY_AGG(b.geoid ORDER BY st_distance(a.geom,
      b.geom) ASC LIMIT 1)[SAFE_OFFSET(0)] as id,
  ARRAY_AGG(st_distance(a.geom,
      b.geom) ORDER BY st_distance(a.geom,
      b.geom) ASC LIMIT 1)[SAFE_OFFSET(0)] as dist

FROM
  table_one a
CROSS JOIN table_two b

GROUP BY a.id
# 29. Calculate the averages of values in neighboring polygons
## BigQuery

WITH
  one AS (
  SELECT
    a.name,
    SUM(b.value) AS total
  FROM
    table a
  LEFT JOIN
    table_2 b
  ON
    ST_Touches(b.geom, a.geom)
  GROUP BY
    a.name)

SELECT
  one.*,
  b.geom
FROM
  one
JOIN
  table b
USING
  (name)
## PostGIS

SELECT
  a.geom,
  a.name,
  SUM(b.value) as total
FROM
  table a
CROSS JOIN LATERAL
  (SELECT value
   FROM table_two
   WHERE 
     ST_Intersects(geom, a.geom)
  ) AS b
GROUP BY a.name, a.geom
# 30. Aggregate points into H3 cells
## BigQuery

WITH
  data AS (
  SELECT
    `carto-un`.carto.H3_FROMGEOGPOINT(geog, 4) AS h3id,
    COUNT(*) AS agg_total
  FROM `cartobq.docs.starbucks_locations_usa`
  GROUP BY h3id
  )
SELECT
  h3id, 
  agg_total,
  `carto-un`.carto.H3_BOUNDARY(h3id) AS geom
FROM
  data
# 31. Rank nearest N neighbors row by row
## PostGIS

SELECT
	a.id, 
  t.geom,
  ST_Distance(t.geom, a.geom) as closest_dist
FROM a
CROSS JOIN LATERAL (
	SELECT *,
	ROW_NUMBER() OVER(ORDER BY a.geom <-> geom) AS rank
  FROM seattle_parks
  LIMIT 5
	) as t
## BigQuery

SELECT
  b.id,
  a.station_id,
  a.rank,
  a.name,
  a.dist
FROM
  table b
LEFT JOIN (
  SELECT
    b.station_id,
    b.name,
    ST_Distance(c.geom, b.geom) AS dist,
    b.id,
    ROW_NUMBER() OVER (PARTITION BY c.id ORDER BY ST_Distance(c.geom, b.geom) ASC) AS rank
  FROM
    table b,
    table_2 h ) a
USING
  (id)
WHERE
  rank < 4
