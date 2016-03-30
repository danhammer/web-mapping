


The objective of this lecture is to learn the basics of SQL (structured query language) through the CartoDB editor.  CartoDB is built on a database called PostgreSQL.  SQL is the languaged used to interact with the database, which is particularly well-suited for GIS because it is a *relational* database.  A relational database stores tabular data (rows and columns) with links or relations easily accessible.  This way, data can be accessed and reassembled without having to reorganize the stored tables.  There is a special database extender for PostgreSQL called PostGIS that makes it easier to make GIS-based queries to the vanilla relational database.  For example, the concept of *closeness* is a location-based relational query.  PostGIS adds support for geographic objects to allow for location queries.  PostGIS allows you to perform geospatial queries such as finding all data points that are within a given radius, the area of polygons in your table, and much more.

**The CartoDB editor simplifies the process of making web-based SQL queries to a PostgreSQL relational database, with the spatial database extender, PostGIS.**

We will be working with the [USGS stream guage data](http://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php) in today's lecture.  

### SQL basics

- Connect the `realstx` dataset, *Realtime US streamflow stations*, which contains streamflow information of over 4,000 stream gage stations.  This is tagged with **Physical datasets**.  Click **New dataset** and search for a tag `:Physical` to find and connect the data.

- Edit the metadata (which is mostly filled out) and set the refresh to **Every day*.  Copy the URL (with the embedded SQL query) above the timing radio buttons for future reference. 

- Visit the [USGS page](http://waterdata.usgs.gov/nwis/rt) to get a sense of the variable meanings.  Specifically, what is a [percentile](http://help.waterdata.usgs.gov/faq/surface-water/what-is-a-percentile?searchterm=percentile)?  What is gage height, also known as [stage](http://help.waterdata.usgs.gov/faq/surface-water/how-to-interpret-gage-height-and-streamflow-values)?

- View just a few, select columns from the table.  
    - What is the default?  
    - Try to **Apply query** with the keyboard shortcuts: `CMD+S` for Mac or `CTRL+S` for PC (basically *saving* the view).  
    - What happens when you clear the view?  
    - Why do you think its called a "view"?

```sql
SELECT the_geom, percentile, stage
FROM realstx
```

- Rename the `stage` variable.  Why might this be useful?

```sql
SELECT stage AS gage_height
FROM realstx
```

- Copy the dataset so that we can edit.  (Note that stitching together the real time datasets and the edited datasets must be done through additional code.)  
    - change the data types for `time` from `string` to `date`.

- Adjust the filters based on `percentile` and `stage`.  
 - What is the SQL query associated with the combined filter?
 - What is the range of values for `floodstage`? 

- Use the filter to start the SQL query to count the number of measurements (rows) that occurred between 1am UTC and 6am UTC today (March 30, 2016).  Ensure you conform to the [proper format](https://en.wikipedia.org/wiki/ISO_8601).  Rename the column view as `num_measurements`
```sql
SELECT COUNT(*) AS num_measurements FROM realstx_copy 
WHERE 
(
    time >= ('2016-03-30T01:00:00Z') 
    AND 
    time <= ('2016-03-30T06:00:00Z')
)
```

- How many watersheds are represented in the data (the variable `huc` is a watershed identifier)?  
    - Find out more information about this watershed by using the USGS link, swapping out the identifier at the apporpriate place in the URL: `http://water.usgs.gov/lookup/getwatershed?01010001`.

```sql
SELECT COUNT(DISTINCT huc) FROM realstx_copy
```

- Select all stations that contain the word `brook` in the name (`staname`).  Browse the table and then view the map.  What happens when you don't select all columns and attempt to map the database view?

```sql
SELECT staname FROM realstx_copy WHERE staname ILIKE '%brook%'
```
```sql
SELECT * FROM realstx_copy WHERE staname ILIKE '%brook%'
```

- Visualize on a map the ten measurements with the highest gage height.  Which state contains the measurement with the *largest* gage height?

```sql
SELECT *
FROM realstx_copy
ORDER BY stage DESC
LIMIT 10
```

### the_geom

Now that we have a handle on some basic SQL, we will shift our focus to two special columns in CartoDB. The first is `the_geom`, which is where some of your geospatial data is stored.  The second is `the_geom_webmercator` which contains all the same points that were in `the_geom`, but projected to Web Mercator, a web-optimized version of the historical Mercator projection. `the_geom_webmercator` is required by CartoDB to display information on your map. It is normally hidden from view because CartoDB updates it in the background so you can work purely in WGS84 (latitude and longitude).  What does this look like?

```sql
SELECT cartodb_id, ST_AsText(the_geom_webmercator) AS the_geom_webmercator
FROM realstx_copy
```

As you can see, the values range from around -20 million meters to +20 million meters in both the N/S and E/W directions because the circumference of the earth is around 40 million meters. This projection takes the furthest North and South to be ± 85.0511°, which allows the earth to be projected as a large square, very convenient for using square tiles with on the web. It excludes the poles, so other projections will have to be used if your data requires them. 

Also note a new type of object appearing in the SQL statement above: `ST_AsText()`. This is a PostGIS function that takes a geometry and returns it in a more readable form.

There are many variants to the common projections, so groups of scientists and engineers got together to create unambiguous designations for projections known as [SRID](https://en.wikipedia.org/wiki/SRID). The two of interest to us are:

- 4326 for [WGS84](https://en.wikipedia.org/wiki/World_Geodetic_System), which defines `the_geom`;
- 3857 for [Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator), which defines `the_geom_webmercator`.

PostGIS will return measurement in the same units as the input projection. Suppose you want to measure the distance between two points stored in `the_geom` that we can call `the_geom_a` and `the_geom_b`. If you input them both into `ST_Distance(the_geom_a, the_geom_b)` the result would come back in units of WGS84. This is not very useful because the answer is in degrees. Instead, we want to measure distance in meters (or kilometers).

You can measure distances (and make many other measurements in PostGIS) using meter units if you run the measurements with data on a spherical globe. That means we can exclude the first version of `ST_Distance()`. Instead, we need to project `the_geom` and our point to PostGIS geography type. We can do this by appending `::geography` to both of them in the function call, as below. Notice that we need to divide the value returned by `ST_Distance()` by 1000 to go from meters to kilometers.

```sql
SELECT
  *,
  ST_Distance(
    the_geom::geography, 
    CDB_LatLng(37.7833,-122.4167)::geography
  ) / 1000 AS dist
FROM
  realstx_copy
```

- What happens when you do this?

```sql
ST_Distance(the_geom, CDB_LatLng(37.7833,-122.4167), true)
```

- Note that you will have to save the result as a new data set in order to share a persistent map.  

### SQL API, view in the browser
- Install a JSON view plugin for your browser.  Search for "json plugin firefox" or whatever browser you use.

```bash
https://danhammergenome.cartodb.com/api/v2/sql?q=SELECT stage, staname FROM realstx_copy ORDER BY stage DESC LIMIT 10
```

## Assignment

1. Create a map of earthquakes in the United States over the past 30 days, using the [USGS earthquake data](http://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php).  Note that the objective is to communicate the spatial information, so think carefully about the design considerations.
    - Color the earthquake points based on distance from USF's campus.  Please think about the colors and number of bins that would make it easy to read the map. 
    - Style and structure the information box with relevant information.
    - Create a text box that titles the map and another with a description and a rating for the difficulty of this assignment (1 easiest -- 10 hardest). 
    - Post this map.  Ensure that it is publicly viewable.
2. Where were the five closest earthquakes to USF in the past 30 days?  Send a URL to return this information in JSON format.

*Much of this lecture is graciously borrowed, stolen, or extended from the CartoDB map academy course.*

