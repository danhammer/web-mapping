This lecture is a gentle introduction to Carto, a web mapping platform that we will rely on for this course.  We will first offer a few examples of what we can do with this web mapping platform, accessing the examples through this GitHub repository.  We will then focus on a particular application: mapping election results for New Hampshire.  Finally, we will discuss the interests of the class, both topical and methodological.  Web mapping is a broad topic which could encompass data science, programming, journalism, design, and many other disciplines across many different topic areas.  We will narrow in on the minimally sufficient set of topics to make the course as broadly interesting as possible.

## Interesting applications

1. San Francisco [Ellis Act Evictions](http://www.antievictionmappingproject.net/ellis.html) (1997-2015)
2. Philadelphia [bike crashes](http://azavea.cartodb.com/viz/16ea1d8e-2481-11e3-aabf-3085a9a956e8/embed_map) (2007-2012)
3. New York City [median days on AirBnB to make rent](https://observatory.cartodb.com/viz/b2cb3416-dbdd-11e5-bbb3-0ea31932ec1d/embed_map), a reaggregation of [this other web map](http://insideairbnb.com/new-york-city/index.html) that relies on Mapbox and D3 (2015-2016)
4. [Global Forest Watch](http://www.globalforestwatch.org) (2000-2015)
5. [Global water detection](http://water.earthgenome.org) and [crop detection](http://danhammer.github.io/crop-website) (1999-2014)
6. [Twitter reaction to Super Bowl](http://srogers.cartodb.com/viz/1b9b0670-8d15-11e3-8ddf-0edd25b1ac90/embed_map) (February 2, 2014)
7. [Tweets mentioning "sunrise"](http://cartodb.s3.amazonaws.com/static_vizz/sunrise.html?title=true&description=true) (April 6, 2014)
8. [Alcatraz escape](https://siggyf.cartodb.com/viz/a3f7aec6-788b-11e4-a565-0e853d047bba/embed_map) (June 11, 1962)

## Guided tutorial: get started with styling

1. **Get the data**.  Navigate to the [Harvard Election Data Archive](https://dataverse.harvard.edu/dataverse/eda) and download [`NH_Shapefile.zip`](https://dl.dropboxusercontent.com/u/5365589/NH_Shapefile.zip), also found [here](https://dataverse.harvard.edu/dataset.xhtml?persistentId=hdl:1902.1/16219).  Click the link and accept the terms of use to save it locally.  The compressed archive (the `.zip` file) has all of the geospatial and metadata associated with a shapefile.

2. **Sign up for CartoDB**.  Navigate to [carto.com](http://www.carto.com).  Click `Sign Up`. (*Note: It may be worth investing in a password manager, like [1Password](https://agilebits.com/onepassword).*)

3. **Connect the dataset**.  Follow the instructions found [here](http://docs.cartodb.com/cartodb-editor/datasets/#connect-dataset) to connect the `NH_Shapefile.zip` dataset.

4. **Style the map**.  The objective is to communicate information that is distributed spatially.  How would we communicate the counties where Democrats constituted more than 50% of the vote in 2008?  What about those counties that are relatively small (with respect to land area)?  

## Guided tutorial: get started with animated maps

1. **Get the data**.  Download active fires from [this repository](https://github.com/danhammer/web-mapping/blob/master/lecture1/active_fires.csv), and add it to Carto as a dataset. Better yet, connect the dataset rather than uploading it from your computer.

2. **Examine the data**. What is the date range of this dataset? (Use the sorting options in the dataset view. We will write SQL queries later to directly answer this questions.) What is the geographic extent?

3. **Create the map**. Click the `CREATE MAP` button.

4. **Style the map**.  Select ANIMATED as the aggregation option for the active fires later, and then select Heatmap as the animated type.  Ensure that the animation occurs on the  We will review the additional options in class.


## Clean up

When you are finished, erase the datasets and maps.  We are using the trial version which has a 250MB storage limit.  You can delete the maps by deleting the datasets.  Click on the datasets at `<username>.carto.com/dashboard/datasets`.

## What do you want to learn?

The topic of web mapping is broad.  We will cover the core skills for functional competence.  These skills can be applied to a number of different topics with a variety of concentrations.  I want to ensure that this course is not only useful, but also interesting to the broadest set of students possible.  With that, consider the following topics and methods that we can ensure we touch in this course.  What would be most interesting?  What are your backgrounds?  How can web mapping be used to further your studies?

Here are a few options for the **methods**:

1. Spatial statistics
2. Data formats
3. API calls
4. Spatial filtering queries (coding)

Here are a few options for the **topics**:

1. Environment
2. Politics
3. Journalism
4. Finance/Economics


