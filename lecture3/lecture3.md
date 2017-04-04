We will review and extend our fluency in SQL. First, we will use the readily available data through the data library, and then, second, we will incorporate external data sources.

#### Filter values based on a separate table

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

- Note that adding conditions or analysis to the query is pretty easy.  The fixed, up-front process of writing the query is tough.  Add a line to show only those earthquakes with a magnitute greater than 2.0.
```sql
INSERT answer INTO here
```

- Another question: Why is [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html) in the PostGIS documentation rather than the standard SQL documentation, like [`IN`](http://www.w3schools.com/sql/sql_in.asp)? 

***

- Import the world airports dataset `ne_10m_airports` tagged as `:Cultural`.  How many airports are in this dataset?  Count using SQL.  Don't add this as a layer yet; remain in the dataset view for the airports data table.
```sql
INSERT answer INTO here
```

- Find all airports that experienced an earthquake within 50 miles in the past day (read: whatever is in the `all_day` dataset).  Use the `ST_DWithin` relationship.  Note that the function requires the distance in meters, so you'll have to use `50 * 1609` as the distance argument (i.e., the number of meters in 50 miles).  Also, remember to use `the_geom_webmercator` to optimize the query.
```sql
INSERT answer INTO here
```

- Create a URL that displays **only the number of airports** in JSON.
```sql
INSERT answer INTO here
```

- Now we can work on the visualization.  Create a map from the airports full data set.  Add the earthquakes layer.  Use [`ST_Buffer`](http://www.postgis.org/docs/ST_Buffer.html) to visualize the "danger zone" of 50 miles around airports. *Note that the buffer `ST_Buffer` will only be **visualized** in the Map view, but it will be available for analysis in the Dataset view.*
```sql
INSERT answer INTO here
```

- Create a map that displays the 50 mile buffer around the airports that suffered a "close" earthquake, along with the earthquakes inside the buffer.  No need to restrict to the US or North America.  Style the map to show the magnitude of the earthquakes.  This is hard.  A straightforward method is to employ two separate queries, one to each layer.

- What is an alternative to the query that relies on `ST_DWithin`?  Adjust the query to replicate the results using `ST_Buffer` and `ST_Contains`.  This is also hard.
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


