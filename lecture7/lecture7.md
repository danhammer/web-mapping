
The objectives of this lecture are:

1. [A review of Medium features to share your maps (and final project)](https://github.com/danhammer/web-mapping/blob/master/lecture7/lecture7.md#medium-to-share-maps)
2. [An actual project that used Carto for irrigation districts in the Central Valley.  Working on `UNION`.](https://github.com/danhammer/web-mapping/blob/master/lecture7/lecture7.md#spatial-calculations)

#### Medium to share maps

There are many services that are now available to easily build simple websites and blogs, including [Wordpress](https://www.wordpress.com), [Squarespace](https://www.squarespace.com/), and [Weebly](https://www.weebly.com/).  We will review how to embed a web map in a blog post, built on [Medium](http://www.medium.com/).  Narrowly, this will be helpful for your final project.  More generally, however, this final step is necessary to realize the value of web mapping.  *Forgive me while I wax philosophical.*  We've learned to present spatial information on a web map, and to distill parceled insights from the underlying data.  We chose CartoDB as the platform of choice because sharing the interactive, aesthetically attentive maps is relatively easy.  Sending a link to a web map may not be sufficient to communicating the mapped information.  It is likely that additional context will be required to reduce the mental overhead to engage with the data.  For this, we need a website, where the maps serve as visualizations of bigger, broader ideas.  All this is to say: Publishing the maps to the web may be just as important as building the maps, since communication is predicated on easy engagement with information.

We will use [Medium](http://www.medium.com/) to create our website for a few reasons.  

1. Embedding maps is easy.
2. Signing up and using the platform is easy.
3. I like the simple aesthetics.

Follow these steps to set up a Medium account.

1. Navigate to [Medium](https://medium.com/).
2. Click **Sign up** to register with an e-mail address.
3. Click **Write a story** and start writing.
4. Publish your story.  You will submit the URL as your final assignment.

Pretty simple.  Probably not worth enumerated steps, but whatever.  The features that will be useful for the final assignment follow:

1. [Embedding code directly](https://webapps.stackexchange.com/questions/66453/how-to-embed-code-snippets-in-medium). There is no syntax highlighting, but it's simple and clean.
2. [Embedding a Gist](https://blog.medium.com/yes-we-get-the-gist-1c2a27cdfc22).  Here, the code you post will be highlighted appropriately, but the interface is not as clean.
3. [Embedding a map](https://help.medium.com/hc/en-us/articles/214981378-Embedding).  Carto is supported by the embedding service that Medium uses.  Click the `<>` icon in the `+` dropdown menu next to the text cursor, paste in the Carto embed URL, and the map will render.

#### Spatial calculations

The objective of this section is to continue to develop some fluency in SQL queries.  I derived this set of problems from a consulting arrangement that I have with the Tulare and Madera irrigation districts.  That is, this is a series of actual questions from the California State Water Resources Control Board.  

- Connect the [county selection table](https://dangeorge.cartodb.com/tables/gsas), derived from the US Counties data where `countyfp = '039'` and `countyfp = '107'` within California, and the California [groundwater basins table](https://dangeorge.carto.com/dataset/i08_b118_ca_groundwaterbasins).

- Create a map of the two counties, adapted to meet the following criteria.  When you are finished, save the data table from the appropriate query, called `gsas_union`.  
    - Dissolve the internal boundaries, i.e., only show the external boundaries of the two counties.  Use the [`ST_Union`](http://postgis.net/docs/ST_Union.html) function.
    - Keep the two counties separate.  There should be **two** rows in the resulting data table.
    - The map interaction should work. Hint: Use the [`row_number()`](http://www.openwinforms.com/row_number_to_sql_select.html) function to create a unique identifier (called `cartodb_id`) after the union.

     ![](http://i.imgur.com/w4oKPYI.png)

    ```sql
    INSERT answer INTO here
    ```
    - Aside: Would this same query work for more than the two supplied counties?  For all US counties?  How would you adjust the code to aggregate the county boundaries to state boundaries?

- We will create a similar map for the groundwater basins *within* the two counties.  Follow the steps to get intermediate information from the data tables.
    - How many separate groundwater basins are at least partially covered by Tulare or Madera county?  Use the `gsas_union` data table you created in the previous step, and count the number of *rows* in the groundwater data (which are technically subbasins).

    ```sql
    INSERT answer INTO here
    ```

    - What is the total area of each county that covers a groundwater basin?  Note that the units won't matter.  We are calculating this number toward a proportion.  Save this two-row table with `countyfp` and `area` variables as a new table called `gbasin_area`.

    ```sql
    INSERT answer INTO here
    ```

    - Join the `gbasin_area` table with the `gsas_union` table on the `countyfp` variable.

    ```sql
    INSERT answer INTO here
    ```

    - Add a column that calculates the area of the counties in the same units as the area calculated before.

    ```sql
    INSERT answer INTO here
    ```

    - Enhance the previous query with a calculation of the proportion of each county that covers a groundwater basin.  You should get around 31% for Tulare and around 37% for Madera.

    ```sql
    INSERT answer INTO here
    ```

    - Create a map that looks like this:

        ![](http://i.imgur.com/nLVgKlP.png)

- This next step may seem redundant.  And, skills-wise, it is redundant.  But this exercise helped me solidify the data manipulations of the previous sections.  It made me feel like a data wizard.  For this, we will use this [sub-table of California watersheds](https://danhammergenome.cartodb.com/tables/watersheds).

    - Add the watersheds to the map, clipped by the county boundaries.  Basically, replicate the following map (note the relevant variable name):

        ![](http://i.imgur.com/nkD6RvU.png)

        ```sql
        INSERT answer INTO here
        ```

    - How many watersheds have some portion that covers the groundwater basins *within* the county boundaries?

#### Thank you!!

It has been such a pleasure to spend time with you all this semester.  There is literally no one in this class that I don't like.  Please keep me posted on what you do with these skills!

