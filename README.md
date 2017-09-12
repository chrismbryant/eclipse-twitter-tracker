# Eclipse Twitter Tracker:

Inspired by [this Google Trends map](https://www.washingtonpost.com/news/wonk/wp/2017/08/01/the-path-of-the-solar-eclipse-is-already-altering-real-world-behavior/?utm_term=.7e148acf90df) of eclipse searches in the week leading up to the ["Great American Eclipse"](https://en.wikipedia.org/wiki/Solar_eclipse_of_August_21,_2017) of 2017, I decided to see if I could track the path of the moon's shadow as it crossed the continental United States by using only Twitter mentions of the eclipse. 

### Intro - *August 18, 2017*:

I am working on writing a program to mine Twitter data from August 21, 2017 (about 9am PDT to 3pm EDT) for mentions of the solar eclipse to see if by using frequency of mentions alone, we can track the path of the moon's shadow throughout the day. Using this data, I hope to generate a changing map of the United States with counties shaded according to proportional tweet volume referencing the eclipse. I will also compare this path to the true path and see if there are any interesting conclusions to be made (Twitter-mentions lagging behind or anticipating the eclipse, Twitter-shadow-area compared to the actual region of totality, tweet content of partial eclipse experiencers vs total eclipse experiencers, etc.)

The rate-limiting of the [Twitter REST API](https://dev.twitter.com/rest/public) is too restricting for the type of large data-acquisition this project requires, so I will try to write up and test a Python script which will use [Twitter's Streaming API](https://dev.twitter.com/streaming/overview) to collect data live during the eclipse. Since I will have no internet access in [Mitchell, OR](https://www.google.com/maps/place/Mitchell+City+Park/@44.5701282,-120.1627358,15z/data=!4m13!1m7!3m6!1s0x54bc0787dd85965b:0x980ceb3d5a2dd9d!2sMitchell,+OR+97750!3b1!8m2!3d44.566525!4d-120.153341!3m4!1s0x54bc07886405da39:0xf3137f0b17ba441d!8m2!3d44.5669464!4d-120.1524881) (where I will be viewing the eclipse from), I will leave a computer running the program at home to automatically collect the relevent Tweets. I will not be able to monitor the progress of the program during collection, so I will program in failsafes to ensure that the total data collected as well as the size of each data file does not get too large. Hopefully, I can get it done before I leave for the airport early tomorrow morning.

### UPDATE - *September 4, 2017*:

Success! The data set ([eclipsefile1.json](https://github.com/chrismbryant/eclipse-twitter-tracker/tree/master/Twitter%20Data)) is smaller than expected, but it looks like the program did what it was supposed to do! 

Now home from my extended eclipse-trip, I am beginning the process of figuring out how to process and map my data. I just posted the [EclipseTracker.ipynb](https://github.com/chrismbryant/eclipse-twitter-tracker/blob/master/EclipseTracker.ipynb) Jupyter Notebook which collected data from Twitter's streaming API while I was away. It appears that roughly 1M tweets in the 6-hour data acquisition period mentioned either the word "eclipse" or "sun," of which 20,933 were geo-tagged. A [quick scatter plot](https://github.com/chrismbryant/eclipse-twitter-tracker/blob/master/Images/myfirsteclipseplot.png) of each of these geo-tagged points on an (X, Y) = (Longitude, Latitude) chart reaveals an image which strikingly appears to match the population density of the United States, but with an added faint trail of points crossing the country in the path of totality.

Moving forward, I will work to bin these points by county (probably using TopoJSON to create the county map) and normalize each measurement by dividing the tweet-count by the county population. Hopefully, this process will accent the path of the eclipse amidst the Twitter noise. 

#### *Some other things took into*:
 * using the [Albers projection](https://en.wikipedia.org/wiki/Albers_projection) to accurately preserve county area;
 * obtaining and plotting the [NASA eclipse path data](https://eclipse.gsfc.nasa.gov/SEpath/SEpath2001/SE2017Aug21Tpath.html); 
 * creating a time-series of images to display shadow movement.  
 
 ### UPDATE - *September 12, 2017*:
 
Since my last update, I have created the beginnings of an interactive [choropleth](https://en.wikipedia.org/wiki/Choropleth_map) of the United States to visualize the Twitter data I collected. My workflow, outlined below, has been divided into two categories: data processing (done mainly in Python) and data visualization (done mainly in JavaScript).  

#### *Data processing*:

Because of my desire to plot my data as a county choropleth, I faced two main challenges: determining which county each tweet was sent from based on its geographic coordinates, and making those county labels compatible with whatever system I would use to plot those counties. Since the geo-tagged Twitter data I collected has relatively inconsistent contents in its ["Place"](https://dev.twitter.com/overview/api/places) field, with geographic identifiers ranging from as precise as a specific street address to as general as the country from which the tweet was sent, I focused on using the bounding box which was present in the same form in every geo-tagged tweet. This box is a collection of 4 [Longitude, Latitude] pairs which define the vertices of the smallest box enclosing the entire specified region. To approximate the bounding box as a single location, for each tweet, I simply averaged the four points so that a bounding box enclosing a city, for example, would become a coordinate at the center of the city. This approach obviously has some issues, such as the fact that a bounding box around Florida becomes a point sitting in the Gulf of Mexico, but rather than fix those issues in some sophisticated way, I decided to just omit those problematic points from my dataset (a decision which I should have realized sooner would cause more problems down the road...[see below]).

I quickly learned that each United States county has a unique 5-digit numerical identifier called a [FIPS county code](https://en.wikipedia.org/wiki/FIPS_county_code), the first 2 digits of which represent the US state (or territory, region, [etc.](https://en.wikipedia.org/wiki/List_of_U.S._state_abbreviations)), and the next 3 of which represent the county or county-equivalent we're identifying within that state. I also determined that to plot the US counties, I would need to use either a [GeoJSON](http://geojson.org/) or [TopoJSON](https://github.com/topojson/topojson) file (which both use the [JSON](http://www.json.org/) file format to encode the shape of geographic objects, each of which can be labeled by their identifying FIPS code).     

The concrete goal now was to convert the list of averaged coordinates obtained from the tweet bounding-boxes into FIPS codes. I played with the idea of creating some sort of algorithm to determine whether a point lies within a [multipolygon](https://gis.stackexchange.com/questions/225368/difference-between-polygon-and-multipolygon), but I instead opted to use the [reverse geocoding API](http://www.datasciencetoolkit.org/developerdocs#coordinates2politics) offered by datasciencetoolkit.org. For each point location I submitted, I received the corresponding county information (easy!). Because the API has some fuzziness built into its algorithm "to combat errors caused by lack of precision," the API returns information on all possible counties the point could lie within if the point happens to be located near a border. 

To create the set of values that I would map to color in my choropleth, I tallied up the number of tweets sent from each county (and if the tweet had multiple associated counties, its one tally would be divided evenly among all those counties; e.g. if the point was near a meeting of 4 counties, each county would receive 0.25 tallies). Tally counts were bound to be higher in places with a higher population, so to get a more useful set of data, I needed to divide the tally count by the county population, giving us an intrinsic *dimensionless* property for plotting. This quantity can be more meaninfully plotted on a choropleth since it is independent of county population or county size. After making a [histogram] of number of counties vs. county population, I determined that county population resembles a [log-normal](https://en.wikipedia.org/wiki/Log-normal_distribution) distribution (which can be visually confirmed by taking the log of county population to reveal a roughly normally distributed number count). This can be understood by the following [Wikipedia quote](https://en.wikipedia.org/wiki/Log-normal_distribution#Occurrence_and_applications) if we consider how a region becomes populated over time: "Many natural growth processes are driven by the accumulation of many small percentage changes. These become additive on a log scale." (NOTE TO SELF: write up a fuller analysis in LaTeX). If tweeting were a statistically random occurrence among the United States population, we should assume that the average tweet count in a county is exactly proportional to that county's population. Dividing the tweet count (per county) by county population would yield a constant. However, since county population fits a log-normal distribution, any anomalies in the tweet/population proportionality will show up as also being log-normally distributed [CITATION NEEDED]. Thus, by taking the logarithm of our tally fractions derived from hopefully anomalous eclipse tweet counts, we should find a normal distribution of data.

We want the data for the choropleth to be normally distributed because this will allow us to obtain good visual contrast across the whole range of data. For example, if we want the choropleth color to range from white to black, we can map the data's mean to the middle gray, two standard deviations below the mean to white, and two standard deviations above the mean to black, encompassing a total of about 95% of the data, smoothly transitioning from the lowest to the highest data value in that range. If we wanted to encompass more data, we could stretch our color range out to 3 standard deviations in each direction, but for just an extra small percentage of included data, this would undesirably reduce the contrast in the central region of data (where most of the data resides). In general, for greater contrast, we shrink the range of data encompassed, and for greater brightness, we increase the center of that encompassed range. I find that a 2-sigma, mean-centered, color mapping provides the best results.    

#### *Data visualization*:

After trying to create my map with a variety of Python tools (e.g. Plotly, Shapely, Basemap) with little success, I decided that using the "Data-Driven Documents" JavaScript Library [D3.js](https://d3js.org/) would be my best path forward. This library is capable of building beautiful highly-customizable interactive visualizations (just look at some of [these examples](https://github.com/d3/d3/wiki/Gallery)!), has great GeoJSON/TopoJSON support, and already has a large amount of material online with viewable source code. The last item on that list has been incredibly important, since, before I started this project, I did not know *any* [JavaScript](https://en.wikipedia.org/wiki/JavaScript), [HTML](https://en.wikipedia.org/wiki/HTML), [CSS](https://en.wikipedia.org/wiki/Cascading_Style_Sheets), or [XML](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)! After a day working through the [Codecademy] (https://www.codecademy.com/learn) courses on JavaScript and HTML, I transitioned into climbing the steep learning curve of D3 by trying to replicate the results of existing data-bound choropleths (which also was made more difficult because while I used [d3.v4](https://github.com/d3/d3/blob/master/CHANGES.md), much of the example material used the older version d3.v3). 

Eventually, thanks in large part to [this tutorial](https://www.youtube.com/watch?v=suNs5p0IxWk) by Venkata Karthik Thota, I got [my choropleth](https://github.com/chrismbryant/eclipse-twitter-tracker/blob/master/Images/tweet_choropleth_1a.png) of Twitter data up and running. 


A fair number of tweets were sent with []

mapshaper.org
