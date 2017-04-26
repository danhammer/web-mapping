
The objectives of this lecture are the following:

1. Review assignment answers
2. Review `UNION` and conditional formatting in CSS
3. Importing Open Street Map data
4. Introduce Medium as a way to publish maps and reports.

#### Assignment 4 answers

| affgeoid         | acq_date   | count |
|------------------|------------|-------|
| 5101300US2001017 | 2016-04-12 | 104   |
| 5101300US2004015 | 2016-04-12 | 83    |
| 5101300US2001197 | 2016-04-12 | 67    |
| 5101300US2001197 | 2016-04-13 | 66    |
| 5101300US2004073 | 2016-04-12 | 66    |
| 5101300US2001161 | 2016-04-12 | 52    |
| 5101300US2002031 | 2016-04-12 | 49    |
| 5101300US2001111 | 2016-04-12 | 47    |
| 5101300US2001127 | 2016-04-12 | 45    |
| 5101300US2004035 | 2016-04-14 | 43    |

#### Lecture
##### Styling maps based on table values

At some point during this exercise, you'll think, "This seems dumb.  Why don't we just add two separate layers?"  The objective is to construct a dataset through SQL and then show how to adjust the CSS to do conditional styling.  So this is more of a heuristic than anything else.

I am using data and motivating text from [this blog post](http://blog.rtwilson.com/john-snows-famous-cholera-analysis-data-in-modern-gis-formats/). In 1854 there was a massive cholera outbreak in Soho, London – in three days over 120 people died from the disease. Famously, John Snow plotted the locations of the deaths on a map and found they clustered around a pump in Broad Street – he suggested that the pump be taken out of service – thus helping to end the epidemic. This then helped him formulate his theory of the spread of cholera by dirty water.

![](https://www.udel.edu/johnmack/frec682/cholera/snow_map_small.png)

This is one of the most famous maps in history, where spatial insight led to a policy change.  The idea is to replicate this map, with conditional CSS queries.

- Upload the `pumps` and `cholera` data sets as zip files (which contain all the shapefile information in the compressed archive) from the [`lecture6`](https://github.com/danhammer/web-mapping/blob/master/lecture6) folder.
- Explore the following query from within the `pumps` data view. What does it do?  What are the data types?
```sql
SELECT cartodb_id, NULL as count, 2 as x, 'pump' as layer
FROM pumps
```
- Navigate to the `cholera` data table and append the `pumps` table using the [`UNION`](http://www.w3schools.com/sql/sql_union.asp) operator.

```sql
INSERT answer INTO here
```

- Ensure that the correct number of pumps and disease incidence are represented in the unioned data table, i.e., replicate the table below with a SQL query.

| layer   | count |
|---------|-------|
| cholera | 250   |
| pump    | 8     |

```sql
INSERT answer INTO here
```

- Add the following lines to the bottom of the CSS editor.  Manually adjust the missing values and play with them.  

```css
INSERT answer INTO here
```
- How does this compare to adding the two separate layers?  Can you think of a previous example where having all the data in a single table with conditional CSS would have been helpful?

- Style the map so that it looks good, looks classic.  Basemaps, colors, etc.  Try to replicate the feel of the original map.

##### Importing Open Street Map data into CartoDB

OpenStreetMap (OSM) allows us to export data on many of the features that make up our cities, including polygons for neighborhoods and cities, roads, and even lampposts. OpenStreetMap data is contributed by a diverse community. It is rich with local knowledge and frequently updated.

- Go to the [OSM website](http://www.openstreetmap.org/#map=19/37.77659/-122.45118).  Note the URL structure, with zoom level, latitude, and longitude.  Export the map exent of something interesting, but ensure that you are at zoom level 19.  Upload the data.  *If you are feeling both bold and lazy, try to figure out how to do this by importing the data via the permalink.*

- Once your data is uploaded, don’t forget to cite the source of the data back to OpenStreetMap in the descripton of your dataset. As part of using OpenStreetMap data, you must give credit to OpenStreetMap per their Copyright and License agreement. You should add an OpenStreetMap link and credit if you don’t plan on using an OSM basemap where the credit is already included by CartoDB.  You can add attribution to OpenStreetMap by editing the metadata associated with your dataset by selecting the “Edit metadata” link in the upper left corner of your dataset in the Data View of the CartoDB Editor.

```
Data © [OpenStreetMap](http://www.openstreetmap.org/copyright) contributors
```

- Delete all datasets aside from the multipolygons and rename this `osm`.  Create a data table with *only* the cartodb ID, the geometry, and the area of the geometry.  Note that you will have to calculate the area based on `the_geom::geography`.  Color the web map by `area` (which will be square feet).

```sql
SELECT cartodb_id, the_geom, ST_AREA(the_geom::geography) FROM osm
```

##### Importing OSM data from Overpass Turbo

I use the Overpass API often in my work.  This is how I get training data for my satellite image algorithms.  Go to [overpass-turbo.eu](http://overpass-turbo.eu/).  You can screen by amenity type.  
- The default Overpass query is below.  Use this as a template in order to find the drinking water amenities in San Francisco.  
- Export these points from Overpass to GeoJSON.  
- Import the resulting file to CartoDB and screen the result to **just San Francisco** as defined by the boundary data set that we used in a previous lecture. 

```js
/*
This is an example Overpass query.
Try it out by pressing the Run button!
You can find more examples with the Load tool.
*/
node
  [amenity=drinking_water]
  ({{bbox}});
out;
```

- Now consider building footprints.
- Use the following query from within Overpass to export the polygons of every polygon (or *way* in OSM-speak) classified as a `school` in OSM in San Francisco. 
- [Here is a list](http://wiki.openstreetmap.org/wiki/Key:amenity) of all OSM amenity tags.  I chose the `school` tag but the categorization is reasonably articulated.  You can work with whatever tag you want, but ensure that it is applied to a polygon (not a line, like a road).  Note that many of these tags are sparse within a map.  You may not get any hits.  This is a growing platform, and the taxonomy is mainly useful for incremental additions.
- We have looked at building footprints before.  Why is Overpass useful in this context?  

```js
way
  [building=yes]
  [amenity=school]
  ({{bbox}});
out;
>;
out skel qt;
```
- How many schools are there in San Francisco?  How many are in each planning neighborhood (also a dataset used in a previous lecture)?

#### Final project

You should begin to work on your final projects.  At least review the available data.  There are some datasets that require you to contact a data administrator for access.  I am not sure how long this process takes, so if you are very excited about a particular dataset then start as early as possible.  Note that you are *not required* to use these (restricted) datasets, or any specific dataset.  Also, please feel free to incorporate other, external data.  Just try to tie it to California, with some thematic focus on environment -- broadly defined.  This can include the usual suspects like water scarcity or biodiversity, but can range to waste management and transport (even transport safety).  

If you end up using the project for work or other courses, please let me know.  I'd love to stay connected to your work.

The final project will be submitted as a Medium post.  Medium supports both code snippets and embeddable maps, both of which are required in the final project.  We will review the features of Medium in the final lecture.  You can jump ahead if you want to get a start on the final write up.  In the meantime, though, here are the requirements of the final project:

1. Three embedded web maps
2. A relevant research question, which can be answered or enhanced with web maps.  
3. Calculations that are the result of at least three *spatial* SQL queries.  These SQL queries should be posted within the text body.

If you just post the bare minimum, you'll get an 80%.  Making the narrative interesting, where maps *actually* enhance the answer, will earn an exceptional grade.
