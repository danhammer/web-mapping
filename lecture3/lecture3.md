We will review and extend our fluency in SQL.  We will use the readily available data through the data library, and then, next, we will incorporate external data sources.  First, though, here are the answers to the assignment:

### Assignment answers

Just about everyone was able to create and style the web map. If you have any further questions, please send me a note!  For the second part ofthe assignment, write your query in the editor to collect the closest 5 earthquakes to USF's campus.  Be sure that you **clear the query** after you finish, just in case you mistakenly link the map to this particular view.  

```sql
SELECT
  mag, latitude, longitude,
  ST_Distance(
    the_geom::geography, 
    CDB_LatLng(37.776429,-122.451287)::geography
  ) / 1000 AS dist
FROM 
  all_day
ORDER BY 
  dist ASC
LIMIT
  5
```

Paste the query after `https://<your_user_name>.cartodb.com/api/v2/sql?q=`.  When you paste this into the browser, you should see the correct result. Note that you can submit the human-readable or the URL-encoded link for full credit, although the first one looks nicer:

```bash
https://dangeorge.cartodb.com/api/v2/sql?q=SELECT mag, latitude, longitude, ST_Distance( the_geom::geography, CDB_LatLng(37.776429,-122.451287)::geography ) / 1000 AS dist FROM all_day order by dist asc limit 5
```

```bash
https://dangeorge.cartodb.com/api/v2/sql?q=SELECT%20mag,%20latitude,%20longitude,%20ST_Distance(%20the_geom::geography,%20CDB_LatLng(37.776429,-122.451287)::geography%20)%20/%201000%20AS%20dist%20FROM%20all_day%20order%20by%20dist%20asc%20limit%205
```

***

#### Sign up for student account

We will need to sign up for Student Developer Packs in order to make use of some of the Carto features in the paid plans.  You will have to sign up for a GitHub handle. Then, go to [this link](https://education.github.com/pack) and enter your information. 

#### Filter values based on attributes in a separate table

Suppose that we want to filter our data based on data from *two separate tables* with possibly *two separate geometry types*.  For example, how do we filter the USGS earthquakes for *just* those within the United States (as opposed to just zooming into the US for the assignment)?  This section will progressively write a query to select points within a polygon.

- Connect the world map of borders through the Data Library. Search for "borders" and you should see two options for world borders. The low-resolution option is sufficient.

- Return to your Datasets pane and connect the near real-time USGS earthquake data through the Data Library. (Search for `earthquakes` and you'll see one matching result.) The table name is `all_day`. The variable names should, roughly, match the columns in Assignment 1.

- Filter the earthquakes in `all_day` to just those within the United States.  
  - We need to refer to two tables in the same query. Experiment with the following two queries. What do they do?
  ```sql
  SELECT
    points.*
  FROM 
    all_day as points, world_borders as polygons
  ```
  ```sql
  SELECT
    polygons.*
  FROM 
    all_day as points, world_borders as polygons
  ```
  - We are only interested in the geometries in the `polygons` layer, where `name = 'United States'`.  Without a proper prefix, however, the column `name` may be ambiguous.
  ```sql
  WHERE
    polygons.name = 'United States'
  ```
  - Finally, we need only the `points` that are contained in the new polygon layer -- with a single geometry of the United States.  For this, we use the PostGIS function [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html).  What happens if you reverse the arguments inside `ST_Contains()`.
  ```sql
  AND
    ST_Contains(
      polygons.the_geom, points.the_geom
    )
  ```
  - Assemble the previous components into a single query and preview the query result.  You don't need to create a map.
  ```sql
  INSERT answer INTO here
  ```

- Note that adding conditions or analysis to the query is pretty easy.  The fixed, up-front process of writing the query is tough.  Add a line to show only those earthquakes with a magnitute greater than 2.0.
```sql
INSERT answer INTO here
```

- Another question: Why is [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html) in the PostGIS documentation rather than the standard SQL documentation, like [`IN`](http://www.w3schools.com/sql/sql_in.asp)? 

#### Filter values based on spatial relationship to elements in another table

- Connect the world airports dataset `ne_10m_airports`.  How many airports are in this dataset?  Count using SQL.
```sql
INSERT answer INTO here
```

- Find all airports that experienced an earthquake within 50 miles in the past day (read: whatever is in the `all_day` dataset).  Use the [`ST_DWithin`](https://postgis.net/docs/ST_Within.html) function.  Note that the function requires the distance in meters, so you'll have to use `50 * 1609` as the distance argument (i.e., the number of meters in 50 miles).  Be sure, also, to ensure that you are working with the [web mercator](https://en.wikipedia.org/wiki/Web_Mercator) projection by using the `the_geom_webmercator` variable. 
```sql
INSERT answer INTO here
```

- What happens if you use `the_geom` rather than `the_geom_webmercator`? Why does that happen?

- Create a URL that displays **only the number of airports** in JSON.  Name the variable `num_airports`.
```sql
INSERT answer INTO here
```

- Now we can work on the visualization.  Create a map from the filtered airports, which are near earthquakes.  Add a layer of the full earthquakes dataset.  Style the map and zoom into southern California.

##### Alternative query, for practice

- What is an alternative to the query that relies on [`ST_Within`](http://postgis.net/docs/manual-1.4/ST_Within.html)?  Adjust the query to replicate the results using [`ST_Buffer`](http://postgis.net/docs/manual-1.4/ST_Buffer.html) and [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html).  This is hard.  
  - First, we need data from the two tables. Call and appropriately name the tables.
  ```sql
  FROM
    ne_10m_airports AS airports, 
    all_day AS earthquakes
  ```
  - Next, we need to buffer the airports.  This creates a new geometry on the fly, which we can then use with the [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html) function.
  ```sql
  ST_Buffer(
    airports.the_geom_webmercator,
    50*1609
  )
  ```
  - Filter the earthquakes to those within the newly minted layer.  
  ```sql
  WHERE
    ST_Intersects(
      ST_Buffer(
        airports.the_geom_webmercator,
        50*1609
      ),
      earthquakes.the_geom_webmercator
    )
  ```
  - Assemble the query from the components above (and other components not yet written):
  ```sql
  INSERT answer INTO here
  ```

#### Assignment 2

Using the Property Land Use Tax lot Output (PLUTO) data table entitled `mnmappluto` in the CartoDB data library, along with the associated [data dictionary](http://www1.nyc.gov/assets/planning/download/pdf/data-maps/open-data/pluto_datadictionary.pdf), answer the following questions:
  1. How many tax lots are there in Manhattan?  How much area does this cover?
  2. Show the growth of the city.  Post a map that colors the tax properties by the year they were built in roughly 20 year increments (from 1900 through the present). How many lots in Manhattan are owned by New York University? What is their average number of floors?
  3. Write a query that calculates the minimum distance between each lot and public spaces (defined as `landuse = '09'`).  This is hard.  Please submit your best guess if you are unable to figure it out.
  
Submit your answers by following the template below:

**Question 1**
  1. Number of tax lots: X
  2. Total area: X

**Question 2**
  1. Web map link: URL
  2. Lots owned by NYU: X
  3. Avg. number of floors: X

**Question 3**
  1. Query: X
  




