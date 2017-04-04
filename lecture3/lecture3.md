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

#### Data science: tweets in San Francisco

Now answer a question that borders on data science.  The insights from this small example are minimal, but the process is something that can easily yield predictive analytics.  Let's predict the weather using tweets and APIs.  First, connect the following four data sets:

1. San Francisco neighborhoods, called `sf_planning_neighborhoods`.
2. Tweets from March 11, 2016 - March 12, 2016 with the search term **weather**.  It was raining during these days in San Francisco.  Name this data table `twitter_sf_rain`.
3. Tweets from April 1, 2016 - April 2, 2016 with the search term **weather**.  It was very nice outside during these days in San Francisco.  Name this data table `twitter_sf_sun`. 
4. San Francisco building footprints, called `sf_building_footprints`.

- Count the number of tweets about weather in San Francisco for both the rainy and sunny days.  *Note: ensure that you use `the_geom_webmercator` to ensure that the points and polygons equivalently projected.*
```sql
INSERT answer INTO here
```

- Count the number of tweets in San Francisco that were posted inside a building on both rainy and sunny days.
```sql
INSERT answer INTO here
```

- Create two API calls for the rainy day, the URLs, with just the count of tweets (1) inside buildings, and (2) the total for San Francisco.

#### Assignment

Using the Property Land Use Tax lot Output (PLUTO) data table entitled `mnmappluto` in the CartoDB data library, along with the associated [data dictionary](http://www1.nyc.gov/assets/planning/download/pdf/data-maps/open-data/pluto_datadictionary.pdf), answer the following questions:
  1. How many tax lots across New York City are there?  How much area does this cover? Post the answers in the subtitle of a map that visualizes the lots by number of floors.  
  2. Show the growth of the city.  Post a map that colors the tax properties by the year they were built in roughly 20 year increments (from 1900 through the present).  In an info box, also answer the following questions: (1) How many lots are owned by New York University?  (2) What is their average number of floors?
  3. Create a map that shows the lots colored by distance to public spaces.

In all, you will post the links to **three** web maps.


