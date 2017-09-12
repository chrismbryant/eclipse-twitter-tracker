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

Because of my desire to plot my data as a county choropleth, I faced two main challenges: determining which county each tweet was sent from based on its geographic coordinates, and making those county labels compatible with whatever system I would use to plot those counties. I quickly learned that each United States county has a unique 5-digit numerical identifier called a [FIPS county code](https://en.wikipedia.org/wiki/FIPS_county_code), the first 2 digits of which represent the US state, territory, region, [etc.](https://en.wikipedia.org/wiki/List_of_U.S._state_abbreviations), and the next 3 of which represent the county or county-equivalent we're identifying within that region.

GeoJSON/TopoJSON data

#### *Data visualization*:

After trying to create my map with a variety of Python tools (e.g. Plotly, Shapely, Basemap) with little success, I decided that using the "Data-Driven Documents" JavaScript Library [D3.js](https://d3js.org/) would be my best path forward. This library is capable of building beautiful highly-customizable interactive visualizations (just look at some of [these examples](https://github.com/d3/d3/wiki/Gallery)!), has great [GeoJSON](http://geojson.org/)/[TopoJSON](https://github.com/topojson/topojson) support, and already has a large amount of material online with viewable source code. The last item on that list has been incredibly important, since, before I started this project, I did not know *any* [JavaScript](https://en.wikipedia.org/wiki/JavaScript), [HTML](https://en.wikipedia.org/wiki/HTML), [CSS](https://en.wikipedia.org/wiki/Cascading_Style_Sheets), or [XML](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)! After a day working through the [Codecademy] (https://www.codecademy.com/learn) courses on JavaScript and HTML, I transitioned into climbing up the steep learning curve of D3 by trying to replicate the results of existing data-bound choropleths (which was made extra difficult because while I used [d3.v4](https://github.com/d3/d3/blob/master/CHANGES.md), much of the example material used d3.v3). 

Eventually, thanks in large part to [this tutorial](https://www.youtube.com/watch?v=suNs5p0IxWk) by Venkata Karthik Thota, I got my Twitter data choropleth up and running. 

mapshaper.org
