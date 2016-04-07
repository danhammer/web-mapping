Now that you are familiar with the USGS earthquakes data set, we will use this data set as the basis for this lecture.  First some clarifications from the previous lecture and assignment:

1. The SQL query in the URL that you generate must also work in the CartoDB editor for your account.  If the query doesn't run in the editor, then it won't work as an API query either.  For example, if you query a table that doesn't exist in the editor, then it won't work via URL access either.  A workflow that works: test scripts in the editor first, and then use them to construct the URL by pasting them after the `?q=` substring in the URL.

2. I will push the answers to the in-class questions to the Github repository *after* class.  This way, you can get some practice quickly finding the answers online or by turning to your classmates.  

### Assignment answers

- **Web map**.  Connect or load the USGS dataset.   Here is [my map](https://danhammergenome.cartodb.com/viz/a75b0fb8-f9cc-11e5-995a-0e5db1731f59/embed_map):

![](http://i.imgur.com/nOnZBKy.png)

- From the editor, set up a query.  Be sure that you **clear the query** after you finish, just in case you mistakenly link the map to this particular view.  Note that the URL may insert URL encoded characters.  A space, for example, will show up as `%20`.

```sql
SELECT
  mag, latitude, longitude,
  ST_Distance(
    the_geom::geography, 
    CDB_LatLng(37.776429,-122.451287)::geography
  ) / 1000 AS dist
FROM 
all_day
order by dist asc
limit 5
```

Either URL is acceptable, although the first one is more readable.

```bash
https://danhammergenome.cartodb.com/api/v2/sql?q=SELECT mag, latitude, longitude, ST_Distance( the_geom::geography, CDB_LatLng(37.776429,-122.451287)::geography ) / 1000 AS dist FROM all_day order by dist asc limit 5
```

```bash
https://danhammergenome.cartodb.com/api/v2/sql?q=SELECT%20mag,%20latitude,%20longitude,%20ST_Distance(%20the_geom::geography,%20CDB_LatLng(37.776429,-122.451287)::geography%20)%20/%201000%20AS%20dist%20FROM%20all_day%20order%20by%20dist%20asc%20limit%205
```
***
*Note: another way to measure distance is using the `use_spheroid` option within `ST_Distance`.  Instead of casting your results using the `::geometry` method, you can simply set `use_spheroid=true`*:

```sql
SELECT
  mag, latitude, longitude,
  ST_Distance(
    the_geom, 
    CDB_LatLng(37.776429,-122.451287), 
    true
  ) / 1000 AS dist
FROM 
all_day
order by dist asc
limit 5
```

### Lecture

We will review and extend our fluency in SQL. First, we will use the readily available data through the data library, and then, second, we will incorporate external data sources.

#### Earthquakes

Suppose, first, that we want to filter our data based on data from *two separate tables* with possibly *two separate geometry types*.  For example, how do we filter the USGS earthquakes for *just* those within the United States (as opposed to just zooming into the US for the assignment)?

- Connect the near real-time USGS earthquake data through the Data Library.  The table name is `all_day` and the tag is `:Physical`.  The variable names (should) match the headers for Assignment 1.

- Connect the world map of borders, also through the Data Library.  You can use the low- or high-resolution option: `World borders` or `world_borders_hd`, respectively.  Both are tagged with `:Administrative`.

- Add the boundary layer to the earthquake map. Filter the boundaries to *only* show (1) the United States and (2) North America. Note that SQL strings **must** use single-quotes, not double-quotes, e.g., `'USA'` not `"USA"`.  Hint: for one country you can use the `=` operator, but for multiple countries you must use [`IN`](http://www.w3schools.com/sql/sql_in.asp).
```sql
INSERT answer INTO here
```

- There is no interaction between points and polygons at this point.  The visualizations are combined, but not the data.  We are going to create that interaction through the SQL editor. Before proceeding, however, style the points and polygons so that they look like a single map: don't let one geometry dominate the other.

- Consider the `ST_Contains` relationship from PostGIS.  Why is it in PostGIS documentation rather than the standard SQL documentation, like [`IN`](http://www.w3schools.com/sql/sql_in.asp)? 

- The first trick is using *two* tables in one query.  Both tables have to exist on your CartoDB account.  Finish the query below to include **only** USGS earthquakes within the United States (and then North America):

```sql
SELECT  
    points.the_geom_webmercator
FROM
    all_day as points, world_borders_hd as polygons
WHERE 
    polygons.adm0_a3 = 'USA'
...
```

- Experiment with your query.  Rename variables and try to get the same answer.

***

- Import the world airports dataset `ne_10m_airports` tagged as `:Cultural`.  How many airports are in this dataset?  Count using SQL.  Don't add this as a layer yet; remain in the dataset view for the airports data table.
```sql
INSERT answer INTO here
```

- Find all airports that experienced an earthquake within 50 miles in the past day (read: whatever is in the `all_day` dataset).  Use the `ST_DWithin` relationship.  Note that the function requires the distance in meters, so you'll have to use `50 * 1609` as the distance argument (i.e., the number of meters in 50 miles).  Also, remember to use `the_geom_webmercator` to optimize the query.
```sql
INSERT answer INTO here
```

- Adjust the query so that only the count of these earthquakes is shown.
```sql
INSERT answer INTO here
```

- Create a URL that displays **only the number of airports** in JSON.
```sql
INSERT url INTO here
```

- Now we can work on the visualization.  Create a map from the airports full data set.  Add the earthquakes layer.  Use [`ST_Buffer`](http://www.postgis.org/docs/ST_Buffer.html) to visualize the "danger zone" of 50 miles around airports. *Note that the buffer `ST_Buffer` will only be **visualized** in the Map view, but it will be available for analysis in the Dataset view.*
```sql
INSERT answer INTO here
```

- Create a map that displays the 50 mile buffer around the airports that suffered a "close" earthquake, along with the earthquakes inside the buffer.  No need to restrict to the US or North America.  Style the map to show the magnitude of the earthquakes.  This is hard.  A straightforward method is to employ two separate queries, one to each layer.
```sql
INSERT answer INTO here
```

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
```sql
INSERT answer INTO here
```

#### Assignment

Using the Property Land Use Tax lot Output (PLUTO) data table entitled `mnmappluto` in the CartoDB data library, along with the associated [data dictionary](http://www1.nyc.gov/assets/planning/download/pdf/data-maps/open-data/pluto_datadictionary.pdf), answer the following questions:
  1. How many tax lots across New York City are there?  How much area does this cover? Post the answers in the subtitle of a map that visualizes the lots by number of floors.  
  2. Show the growth of the city.  Post a map that colors the tax properties by the year they were built in roughly 20 year increments (from 1900 through the present).  In an info box, also answer the following questions: (1) How many lots are owned by New York University?  (2) What is their average number of floors?
  3. Create a map that shows the lots colored by distance to public spaces.

In all, you will post the links to **three** web maps.


