The objective of this lecture is to review the SQL that we've learned already, adding in basic merge operations.  We will first use the *Merge with dataset* option from within the CartoDB editor and then layer in SQL queries for the instances when the point-and-click option fails.  First, however, we will review the previous assignments.  We will review the grading for [Assignment 1](https://github.com/danhammer/web-mapping/blob/master/lecture2/Lecture2.md#assignment) and the answers for [Assignment 2](https://github.com/danhammer/web-mapping/blob/master/lecture3/lecture3.md#assignment).

A few motivating examples that are especially useful or attractive.

1. [Flint lead testing results](http://michiganradio.org/post/map-take-closer-look-flint-lead-testing-results#stream/0)
2. [United States population map](https://observatory.cartodb.com/viz/582f22f2-d682-11e5-a3bd-0ecfd53eb7d3/embed_map)
3. [Bay Area car crashes](https://team.cartodb.com/u/mamataakella/viz/322d015a-e9fb-11e5-b482-0e31c9be1b51/embed_map)

#### Assignment 1 grades

Manily just dings.  Please be sure to follow instructions exactly.  And **really** make sure that the tables exist.  Out of 100, the current average is 90 with a standard deviation of 5 points (i.e., 66% of the class scored between 85-90 points).  If you would like to resubmit the assignment to *precisely* conform with the instructions, then I will readjust with a 50% penalty.  This means that most can be in A-range with readjustments.  Here are a few of the common mistakes:

1. The web map URL was submitted with `public_map` instead of the requested `embed_map`.  No points were deducted for this; but please resubmit with the proper link.
2. The table referenced in the API call no longer exists, most likely deleted.  Make sure that the query actually works.
3. Some URLs have some weird text inserted **(Links to an external site.)**.  Be sure that the submission actually works.
4. The earthquakes in the first web map are not colored by distance, but rather magnitude or not at all.  

#### Assignment 2 answers

The objective of this assignment was to replicate some of the maps in the [PLUTO Data Tour](http://andrewxhill.com/cartodb-examples/scroll-story/pluto), created by CartoDB's Chief Science Officer, Andrew Hill.  All answers rely exclusively on the `mnmappluto` data set, available through CartoDB's Data Library.

**Question 1.** The first question asks you to count the number of tax lots in the dataset.  Note that `mnmappluto` covers *only* Manhattan, whereas the Data Tour covers all five boroughs.  The exact answers will be different, but the process to find the answers is roughly identical.  Count the number of tax lots.
```sql
SELECT COUNT(*) FROM mnmappluto           # VALID
SELECT COUNT(cartodb_id) FROM mnmappluto  # VALID
SELECT COUNT(address) FROM mnmappluto     # INVALID
```
Note that if you count rows for all columns or columns with guaranteed non-null values, then the answer is 42,890.  If, however, you count the values in a column with null values, then the count will go down -- the default counts non-null values.  Each row definitely has a `cartodb_id` so it's best to count on this variable.

Since it is unclear whether it's meaningful to have a public space with number of floors, maybe it's best to filter these out.  Again, this is not (strictly speaking) part of the assignment, but it will be part of the final project -- effectively communicate a story with maps.  [**This map**](https://danhammergenome.cartodb.com/viz/70ffb056-0069-11e6-ba6e-0e674067d321/embed_map) is visualized with the following query.
```sql
SELECT * FROM mnmappluto WHERE zonedist1 NOT ILIKE 'PARK%'
```

**Question 2.** Ensure that you return to the original data set, rather than the Map View used in the previous section.  [**Visualize a new map**](https://danhammergenome.cartodb.com/viz/a7dcbd6c-006e-11e6-b495-0e787de82d45/embed_map).  Adjust the CSS to conform to 20 year increments.  You can also create a series of maps, like the Data Tour, to 

```css
/** choropleth visualization */

#mnmappluto{
  polygon-fill: #FFFFB2;
  polygon-opacity: 1;
  line-color: #FFF;
  line-width: 0;
  line-opacity: 1;
}
#mnmappluto [ yearbuilt <= 2020] {
   polygon-fill: #B10026;
}
#mnmappluto [ yearbuilt <= 2000] {
   polygon-fill: #E31A1C;
}
#mnmappluto [ yearbuilt <= 1980] {
   polygon-fill: #FC4E2A;
}
#mnmappluto [ yearbuilt <= 1960] {
   polygon-fill: #FD8D3C;
}
#mnmappluto [ yearbuilt <= 1940] {
   polygon-fill: #FEB24C;
}
#mnmappluto [ yearbuilt <= 1920] {
   polygon-fill: #FED976;
}
#mnmappluto [ yearbuilt <= 1900] {
   polygon-fill: #FFFFB2;
}
```

Count the number of buildings owned by New York University (92) and determine the average number of floors (8.44) with the following separate queries:
```sql
SELECT COUNT(*) 
FROM mnmappluto 
WHERE ownername ILIKE '%new york university%'
```
```sql
SELECT AVG(numfloors) 
FROM mnmappluto 
WHERE ownername ILIKE '%new york university%'
```

#### Data set merges

First, we'll use the built-in, point-and-click merge for a *column join* to add attributes from `world_borders` to the `populated_places` table.

1. Connect the `world_borders` dataset (not the high definition version).

2. Connect the `Populated places` dataset.  Rename to `populated_places` in the Data View.  Note that you will have to change the sync option to **Never** in order to change the name.  Also adjust the metadata to note these changes (this is just good practice).

3. We will be merging on column value, or the ISO code for each row.  Navigate back to `world_borders`.  Select *Merge with dataset* from the drop-down menu labeled **Edit**.

4. Select *Column join*.  Merge the `iso2` code from `world_borders` with `adm0_a3` from the populated places table.  

5. Click *MERGE DATASETS*.  A new dataset is created and displayed in the Data View.  It is especially important to edit metadata after merges, given default table names.  What happened?  How many unique rows are in this dataset?  Note this number for later.

-----
Next, we will use a spatial join, where attributes are joined based on their location rather than the value of a specified variable.  This se The selection of variables now determines the attributes in the merged data set.  Use the **Spatial join** option to answer these questions.  (There are other ways, which we'll get to.)

1. How many populated places are in Brazil?  
2. Create a chloropleth map of the number of populated places for each country.  Adjust the infowindow to explore the number of populated places for all countries (not just Brazil).
3. Assume that the `pop_max` is the population of the populated places (in 10s of people).  What is the average population of the populated places in Japan?
4. What is the total 2005 world population?
5. How many megacities are their in the United States?  In China? In Mexico?

An unguided series of questions:

1. How many airports are there in Argentina?  
2. How many airports are there in countries with names that begin with the letter "B"?
3. How many **major** airports are there in the United States?

-----
The CartoDB *Merges* queries rely on the SQL `JOIN` function. There are different types of `JOINs`:

- `INNER JOIN`: Returns all rows when there is at least one match in BOTH tables
- `LEFT JOIN`: Return all rows from the left table, and the matched rows from the right table
- `RIGHT JOIN`: Return all rows from the right table, and the matched rows from the left table
- `FULL JOIN`: Return all rows when there is a match in ONE of the tables

The basic syntax for a `JOIN` follows.  Note that the variable and table names are made up.  You will have to change everything aside from the SQL operations and their order.  The objective is *just* to provide a template.

```sql
SELECT original.cartodb_id, original.the_geom, original.variable, target.variable
FROM original
INNER JOIN target
ON original.idx=target.idx
```

1. Without relying on the point-and-click Merge option in CartoDB, merge the 2005 national population into the populated places data set. 
```sql
INSERT answer INTO here
```

Now, upload the Panama Papers data set.  This data set contains information on the number beneficiaries, clients, companies, and shareholders listed in the incriminating Panama Papers.  When you upload this information, CartoDB will try to intelligently geolocate the information.  We will ignore this, since it is imperfect.  We can do better with the raw SQL.  

1. Connect the `panama_papers.csv` data set by first downloading the CSV from GitHub.
2. Merge the `panama_papers` table with the `world_borders` table.  Be sure to include at least the `beneficiaries` variable from the `panama_papers` table.
3. Create a map where each country is colored by the number of beneficiaries.  Be sure to specify the merge so that the geometry for each country is displayed, *even if there are no beneficiaries listed, i.e., null values.*
```sql
INSERT answer INTO here
```
1. Explore the difference between the different types of merges by counting the number of rows in the merge, without creating a new table for each new merged table.  This will require a **sub-query**.  You are, in effect, creating a table on the fly, and counting the rows from that. You will most likely encounter an error message that reads *subquery in FROM must have an alias*.  Think about this and fix the nested query.
```sql
INSERT answer INTO here
```

----

Keep the table available.  We will now look at the `GROUP BY` clause to aggregate by certain values in the table.  Suppose, for example, you wanted to count the beneficiaries by region in the new, merged data set.  The template follows.

```sql
SELECT column1, column2
FROM table_name
WHERE [ conditions ]
GROUP BY column1, column2
ORDER BY column1, column2
```
1.  Now, you are allowed to create a new data set with the beneficiearies for each country.  
2.  Use this new data set to count the number of beneficiaries by region.  Order by the number of beneficiaries, from most to least.

*Note that you will ultimately need to use the `GROUP BY` clause to effectively do Question 3, Assignment 2.*

#### Assignment 3

1. Create a web map that shows the ratio of Panama Paper beneficiaries to regional airports *for each country*.
2. Try to answer Question 3 of the previous assignment for just the first thousand entries in the PLUTO data table (i.e., the records with a CartoDB ID less than 1000).  Navigate to the lower east side to create a web map, with each record colored by distance to public space.  I would encourage you to create two separate tables, one for each sample (the first 1000 records *and* the appropriate public spaces data table).  The following SQL is a hint: the last few lines of the necessary query.
```sql
WHERE 
original.cartodb_id < 1000
AND
original.cartodb_id < modified.cartodb_id
GROUP BY 
original.cartodb_id,
original.the_geom_webmercator
```