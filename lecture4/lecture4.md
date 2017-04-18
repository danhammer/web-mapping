The objective of this lecture is to review the SQL that we've learned already, adding in basic merge operations. Additionally, we will start to think about the final project, which will be published as a webpage.  Here are a few motivating examples that may be helpful:

1. [Flint lead testing results](http://michiganradio.org/post/map-take-closer-look-flint-lead-testing-results#stream/0)
2. [United States population map](https://observatory.cartodb.com/viz/582f22f2-d682-11e5-a3bd-0ecfd53eb7d3/embed_map)
3. [Bay Area car crashes](https://team.cartodb.com/u/mamataakella/viz/322d015a-e9fb-11e5-b482-0e31c9be1b51/embed_map)

### Chloropleth, quickly

We will generate a series of chloropleth maps, relying on spatial and non-spatial joins of two datasets.  A spatial join relies on the spatial relationship between the two datasets.  For example, suppose you have a dataset of U.S. wind turbine locations that *do not have the state as a column*.  How do you add that column?  How do you count the number of wind turbines by state?  A non-spatial join is based on a common column across the two tables.  Suppose, for example, that you have a dataset of population centers that have a country column.  How do you display the number of population centers in each country on a map?  This is basically the same as spatial joins for our purposes, except we don't join using a PostGIS statement.  

#### Non-spatial Joins

There are four basic types of joins in SQL:

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

##### Example: population centers

Start by making a chloropleth map of population centers by country.

- Connect the `world_borders` dataset (not the high definition version) and the most populated places dataset (`ne_10m_populated_places_simple`).
- Join the two datasets with an `LEFT JOIN` on `iso3` in `world_borders` and `adm0_a3` in `ne_10m_populated_places_simple`.

```sql
INSERT answer INTO here
```

- **Tricky question**.  Without saving the new view, use *just* the SQL to create a count of populated places by country.  Preserve the country name along with any other required variables to make the web map (i.e., `the_geom_webmercator` and `cartodb_id`).  You will have to write this SQL as a nested query, i.e., you will have two `SELECT` statements within the same, lengthy query.

```sql
INSERT answer INTO here
```

- Create a copy of this super table.  And create a chloropleth of that map, where the country polygons are colored by the number of population centers within the country borders.

##### Example: satellites

I have cleaned a dataset of all active satellites in orbit.  The dataset is called `active_satellites.csv`.  The objective of this subsection is to create a chloropleth map of the satellites' countries of origin.

- Connect the `active_satellites` table.
- Adjust the queries from the previous section to join `name` in `world_borders` with `country` in `active_satellites`.  Use an `INNER JOIN`.  Why an `INNER JOIN`?  Remember to make a copy of the table before creating the map.  

```sql
INSERT answer INTO here
```

- Adjust the query in the previous section to create a chloropleth map of just the earth observing satellites.  

```sql
INSERT answer INTO here
```

#### Spatial Joins

What happens if you don't have a column with a country code or name?  The join can't be indexed by a common attribute value, but it might be indexed by location.  

##### Example: turbines by county

We can create a chloropleth map of turbines by county.  The turbines dataset actually does have the names of counties in the table; but ignore that for the time being.  We can use that column as a check on our spatial join.

- Connect [`turbines`](https://dangeorge.carto.com/dataset/turbines) and the table of "USA counties" (`cb_2013_us_county_500k`).
- Use the [`ST_Contains`](http://postgis.net/docs/manual-1.4/ST_Contains.html) relationship to join the two tables (spatially) to get the count of turbines within each U.S. county.  Which county has the most turbines?

```sql
INSERT answer INTO here
```

- Create a map of U.S. counties with over 100 turbines, colored by number of turbines.

```sql
INSERT answer INTO here
```

##### Example: farmers' markets by county

- Connect `farmers_mkts.csv` from this repository.
- Create a map of U.S. counties colored by number of farmers markets that take credit cards.

```sql
INSERT answer INTO here
```

### Assignment 3

- Use the `panama_papers` dataset to recreate *exactly* the web map in [this post](https://www.cgdev.org/blog/panama-papers-and-correlates-hidden-activity).  
- Create another map colored by the ratio of number of beneficiaries to number of airports in the country.
