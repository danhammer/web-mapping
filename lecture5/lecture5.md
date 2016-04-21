

We will begin to assemble all of the functions from previous lectures into a sensible data narrative.  As you go through the (leading) questions, think about ways in which you'd supplement the data visualizations with insight.  Think of suggestions about how this might be used in a newspaper article or an academic publication.  

There are gaps in this lecture that are intentional.  If you have questions about previous assignments or embedded queries, then please don't hesitate to ask.

----
#### Assignment 3 answers

**Question 1**.  Merge the `panama_papers` data table with `world_borders`.  This will tie the number of beneficiaries to the country border geometries.  Then spatially join the new data table, called `world_beneficiaries` with the airport count.  Call this **new** data table `beneficiaries_airports` and rename the variable `intersect_count` to `num_airports`.  This data table should have the name of the country, beneficiaries, and a region identifier.  

```sql
SELECT region, SUM(num_airports) AS num
FROM beneficiaries_airports
GROUP BY region
ORDER BY num DESC
```

Create a new data table from this query that yields the number of regional airports.  *Note that you can combine all these queries; but for the sake of simplicity and exposition, it is preferable to create new tables.*  Call this `regional_airports`.  Now the final merge and calculations, all at once:

```sql
SELECT 
    cartodb_id, 
    the_geom_webmercator, 
    name, 
    (beneficiaries/num) AS ratio
FROM (
    SELECT original.*, target.num
    FROM beneficiaries_airports AS original
    INNER JOIN regional_airports AS target
    ON original.region=target.region
) AS final_table
WHERE num > 0
```

Color the map by ratio.  Note that this data visualization is, in fact, way more advanced than even the map provided in [this blog post](http://www.cgdev.org/blog/panama-papers-and-correlates-hidden-activity) by one of the top economic think tanks in the world.

**Question 2**. First split the dataset into parks and not parks.  I would save these as separate tables, `mnmappluto_park` and `mnmappluto_not_park`, using the following two clauses:

```sql
SELECT * FROM mnmappluto WHERE zonedist1 ILIKE 'PARK%'
```
```sql
SELECT * FROM mnmappluto WHERE zonedist1 NOT ILIKE 'PARK%'
```

You can split the dataset into two parts using other definitions of public vs. nonpublic landuse, e.g., using the `landuse` variable.  Add both layers to the same map.  Color the public spaces using a "simple" coloring, maybe all green.  Color the nonpublic spaces according to the *minimum* distance.

```sql
SELECT 
mnmappluto_not_park.*,
MIN(
  ST_Distance(
    mnmappluto_not_park.the_geom,
    mnmappluto_park.the_geom
  )
) AS dist
FROM mnmappluto_not_park, mnmappluto_park
GROUP BY mnmappluto_not_park.cartodb_id
```

Note that you will have to use the `MIN` operator to create a variable.  If you try this without a `GROUP BY` clause, you will get an error.  Effectively, you are pairwise comparing each nonpublic lot to each public lot.  Then you are collapsing this implicit array into a single value according to the minimum value.  This is where the `GROUP BY` clause comes in; and why it's so difficult, conceptually.  The quick rule-of-thumb is that whenever you are using a function to create a new variable on the fly, you will have to use a `GROUP BY` clause -- unless you want to collapse the *entire* column.  You would only do this for a basic `COUNT` or a simple `AVG` across an entire column.  

Note also that if you *do not* create two separate data tables, then you may run into computation problems.  You may fall into a weird recursive loop.  There are ways to get around this, like adding the conditional to the query in order to prevent duplicate comparisons.

```sql
WHERE mnmappluto_not_park.cartodb_id < mnmappluto_park.cartodb_id
```

All that said, here is [**my version**](https://danhammergenome.cartodb.com/viz/f74514bc-028f-11e6-90bf-0ecfd53eb7d3/embed_map).

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

1. Post the **single** SQL query to find the ten counties and days with the most fires detected by NASA satellites in the past seven days.  To be clear, there should be just one SQL query.  A county on one day is different from a county on another day.  You are looking for the ten county-days with the most fires (a county may be listed twice if it contained a lot of fires on two days).  Connect the fires data table from [here](https://danhammergenome.cartodb.com/tables/fires/public).  You can connect directly via the **Create Map** button.  Use the `cb_2013_us_county_500k` as the counties data table, found within the Data Library.
2. Create an animated web map that loops once every 7 seconds, one second for each day represented in the fires data set.  Spend some time making this look good.  
