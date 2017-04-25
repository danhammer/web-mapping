

We will begin to assemble all of the functions from previous lectures into a sensible data narrative.  As you go through the (leading) questions, think about ways in which you'd supplement the data visualizations with insight.  Think of suggestions about how this might be used in a newspaper article or an academic publication.  

----
#### Assignment 3 answers

**Question 1**.  
- Create a table with the sum of "links" in the table.  Save the table.  You can call it by the default name, `panama_papers_copy`. 
```sql
SELECT 
    cartodb_id, 
    iso, 
    the_geom_webmercator, 
    SUM(beneficiaries + clients + shareholders + companies) as total
FROM 
    panama_papers
GROUP BY 
    cartodb_id
```
- Merge the `panama_papers_copy` table with `world_borders` to link the number of beneficiaries to the country border geometries. Save this new table as `world_links`, create a map, and style the chloropleth according to the number of beneficiaries.  A bonus is to directly use the colors seen in the [reference map published by CGD](https://www.cgdev.org/blog/panama-papers-and-correlates-hidden-activity) by editing the CartoCSS.
```sql
SELECT 
    original.cartodb_id,
    original.the_geom_webmercator,
    target.total
FROM
    world_borders as original
INNER JOIN 
    panama_papers_copy as target
ON original.iso2=target.iso
```
**Question 2**.
- Spatially join the new data table, which I've called `world_links`, to the airport count.  Create a new table called `airport_total` from the query.

```sql
SELECT cartodb_id, the_geom_webmercator, total, count(cartodb_id)
FROM 
  (
    SELECT original.cartodb_id, original.the_geom_webmercator, original.total
    FROM world_links as original
    INNER JOIN ne_10m_airports as target
    ON ST_CONTAINS(original.the_geom_webmercator, target.the_geom_webmercator)
  ) as temp_table
GROUP BY cartodb_id, the_geom_webmercator, total
```
- Apply the final query to get the ratio of links to airports, noting that you can't divide by zero. Color the map by ratio.  Note that this data visualization is, in fact, way more advanced than even the map provided in [this blog post](http://www.cgdev.org/blog/panama-papers-and-correlates-hidden-activity) by one of the top economic think tanks in the world.
```sql
SELECT 
    cartodb_id, 
    the_geom_webmercator, 
    (total/count) AS ratio
FROM airport_total
WHERE count > 0
```

**Question 3 (BONUS)**. 

- The objective of this question was to combine the minimum, public space queries into a single query.  Note that the previous two questions can also be combined into a single, super long query.
```sql
SELECT 
	original.cartodb_id, 
    original.the_geom_webmercator,
    MIN(ST_Distance(
        original.the_geom, 
        modified.the_geom,
        true
    )) AS dist 
FROM 
	mnmappluto AS original, 
    (
      SELECT * FROM mnmappluto WHERE landuse = '09'
    ) AS modified
GROUP BY original.cartodb_id
LIMIT 10
```
----

#### Lecture-Lab

1. Go to data.gov and download the data set [Birds - Breeding Survey Routes [ds60]](http://catalog.data.gov/dataset/birds-breeding-survey-routes-ds60). This data set provides access to information gathered on annual breeding bird surveys in California using a map layer developed by the Department. This data layer links to the breeding birds survey information that has been gathered from observers, compiled, and made available over the internet by the Migratory Bird Research Program, Patuxent Wildlife Research Center of the U.S. Geological Survey. The Breeding Bird Survey web site can be reached directly [here](http://www.pwrc.usgs.gov/bbs). A more complete description of the breeding bird survey program is available [here](http://www.mbr-pwrc.usgs.gov/bbs/genintro.html and http://www.pwrc.usgs.gov/bbs/about).
2. Download [Onshore Industrial Wind Turbine Locations for the United States to March 2014](http://catalog.data.gov/dataset/onshore-industrial-wind-turbine-locations-for-the-united-states-to-march-201453ff7). This data set provides industrial-scale onshore wind turbine locations, corresponding facility information, and turbine technical specifications, in the United States to March 2014. The database has nearly 49,000 wind turbine records that have been collected, digitized, locationally verified, and internally quality assured and quality controlled. Turbines from the Federal Aviation Administration Digital Obstacle File, product date March 2, 2014, were used as the primary source of turbine data points.
3. Download the most recent [Climate Prediction Center (CPC) Monthly Drought Outlook (MDO)](http://catalog.data.gov/dataset/climate-prediction-center-cpc-monthly-drought-outlook-mdo). The map shows where current drought areas are expected to improve, be removed, or persist with intensity, as well as new areas where drought may develop, at the end of the forecast period.
4. Connect the National Parks boundaries from the CartoDB Data Library.  This table includes Parks, Monuments, Seashores, and Recreation Areas.
5. Upload the four datasets.  Name them `bird_routes`, `turbines`, `drought`, and `parks`.

Answer the following questions:

- How many wind turbines are within 2 miles of USGS bird routes in California?  Display these turbines on a map with *only* the affected bird routes.  Style the map so that you can adequately see the point of this.  Also, for kicks, turn on the satellite image basemap to see the actual turbines from space.  **Note**: to display the turbines within two miles of bird routes *and* the bird routes within two miles of turbines, you will have to apply the `ST_DWithin` query within the SQL editoar for both layers on the same map.

```sql
SELECT COUNT(*) 
FROM (
    SELECT
        turbines.*
    FROM
        turbines,
        bird_routes
    WHERE
        ST_DWithin(
            turbines.the_geom_webmercator,
            bird_routes.the_geom_webmercator,
            2*1609
        )
) AS counter
```


- Suppose that the required buffer is a function of the `stratum` variable in the `bird_routes` table.  Specifically, suppose that the original 2 mile buffer is scaled by `stratum`/100.  Create a web map of these buffers that includes just the CartoDB ID, the stratum, and the geometry (for mapping).  You will run into a problem when you directly try to divide `stratum` by one hundred.  You will first have to [`CAST`](https://www.1keydata.com/sql/sql-cast.html) the stratum as a `FLOAT` variable.

```sql
SELECT
    bird_routes.cartodb_id,
    ST_Buffer(
      bird_routes.the_geom_webmercator,
      2*1609
    ) AS the_geom_webmercator
FROM
    turbines,
    bird_routes
WHERE
    ST_DWithin(
        turbines.the_geom_webmercator,
        bird_routes.the_geom_webmercator,
        2*1609
    )
```

```sql
SELECT 
cartodb_id, 
stratum,
ST_Buffer(
    the_geom_webmercator,
    CAST(stratum AS FLOAT)/50*1609
) AS the_geom_webmercator
FROM bird_routes
```

- I am a birder.  I'm not, but I could be.  Suppose I want to go to a US National Park (not a National Preserve or National Monument) for bird watching.  Which National Parks are along bird routes in California?

```sql
SELECT
    nps_boundary.*
FROM
    nps_boundary,
    bird_routes
WHERE
    ST_Intersects(
        nps_boundary.the_geom_webmercator,
        bird_routes.the_geom_webmercator
    )
```

- Suppose that I now only want to visit national parks with bird routes *with persisting drought*.  You know, the golden grass.  Use the [most recent drought data](http://www.cpc.ncep.noaa.gov/products/GIS/GIS_DATA/droughtlook/index.php) from NOAA.  This is harder than it seems, given the poor data quality of the download.


----
#### Assignment 4

This is the **last** assignment before the final write-up!  

1. Use the [Active Fire Data](https://earthdata.nasa.gov/earth-observation-data/near-real-time/firms/active-fire-data) tool to download active fires for the "USA (Conterminous) and Hawaii" in the past 7 days from the MODIS 1km Sensor.  Post the **single** SQL query to find the ten counties and days with the most fires detected by NASA satellites in the past seven days. To be clear, a county on one day should be treated differently than a county on another day.  You are looking for the ten county-days with the most fires (a county may be listed twice if it contained a lot of fires on two days). 

2. Create an animated web map that loops every 7 seconds, one second for each day represented in the fires data set.  Spend some time making this look good.  Post the CartoCSS along with the web map URL.  
