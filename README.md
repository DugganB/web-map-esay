# COVID-19 Temporal Web Map Analysis 
By: Duggan Burke
GEOG 458
June 6, 2020

## Intro

This map was created by me, I will be giving an overview of the map and some of the goals I set out to accomplish during this project. I am in no way affiliated with any biased groups besides being a student at the UW. The web map can be found at [covid.dugganb.dev](www.covid.dugganb.dev).

This project serves as a informative tool so that people can make their own decisions about COVID-19 and see how the disease has spread through the country. No other web maps have been made for displaying the county level data over time. Many such maps have been made which show live information to the viewer, but not the past information. With a disease you need to understand how it spreads and it's history to make effective policy choices. 

Below you can see a screenshot of the interactive web map. On the bottom is a slider which displays the date the choropleth is display the statistics of. On the left side is a panel which displays radio buttons which allow the user to change which statistic is being displayed. The user can click on a specific county to view all the statistics for that date for that county. The right panel displays all the statistics of the the selected county.

.

The main audience for this map is the average person. I built this to allow myself and others to see how the virus has spread through the country. Every day there are new news stories and new data or research coming out around COVID-19. This map is an attempt to allow anyone to understand a large amount of information and come up with their own findings and ideas.

## Architecture

This application has a script which takes in the data and outputs it into a final form for the web application. The web application is a React app. The app is bundled with all of the data so there is no downloading of data after the initial load. 

The data is downloaded, onto the computer from which the script is run, from the [New York Times GitHub](https://www.kaggle.com/fireballbyedimyrnmom/us-counties-covid-19-dataset) and converted to JSON for easy processing. The script is run using node which outputs a final data set for the map that is delivered in the application. When the repo is committed and pushed to GitHub, Netlify kicks off a build and then deploys the site upon completion of the build. 

The whole project uses the following libraries/services:
 - ReactJS
 - Leaflet
 - Node
 - Netlify

The React application imports the JSON data in the following manner:

```javascript
    import USCounties from "../data/counties.json";
    import ProcessedData from "../data/processed-data-per-county.json";
```

`USCounties` is the polygons which represent the actual counties to be displayed. This is stored in a GEOJSON format. This county polygon data was downloaded from the US Census bureau. The initial file size was around 202MB, which is way too big for a web application. I tried doing some basic simplification of the polygons, but upon inspection found that there were these slivers of empty space because the simplification was not topologically aware. After doing some research I found that QGIS with GRASS Generalization was a good way to do polygon simplification while keeping topology. I set the tolerance on GRASS Generalization to 0.01 degrees and ran the script, this resulted in a file size of about 6MB. This is much better for a web application and would not take too long to download while still having a good amount of details. In order for Leaflet to not show it's own slivers inbetween polygons as it ran its own simplification upon zoom in/out the smoothFactor had to be set as seen below.

```javascript
    <GeoJSON
	  data={USCounties}
	  style={(layer) => this.style(layer)}
	  smoothFactor={0.25}
	  stroke={false}
    />
```

The `ProcessedData` object contains all of the computed statistics for every date and every county. The data has been constructed in such a way to allow for efficient access by the client so that it is not sluggish and eat up lots of processing power. The structure of the data is:

```javascript
    {
      Date1: {
	      CountyID: {county stats},
	      ...
      },
      Date2: {
	      CountyID: {county stats},
	      ...
      },
      ...
    }
```

This allows the application to simply get all the county data with the selected date and then select the specific county statistics by the countyID which is on the GEOJSON polygon data for the counties. This use of nested maps allows for very fast retrieval of information. There are about 3000 counties in the US, so efficiency with rendering all of the data is important.

Leaflet is used for the main map, specifically react-leaflet. This is just a React wrapper around the Leaflet library to make React development with Leaflet easier. Leaftlet makes it easy to display GEOJSON data and use custom tile layers.

The tile layer is a custom generated tile layer using Mapbox. I took one of their default tile layers and tweaked the colors to match my color scheme as well as reduce the amount of information shown by the tile layer. By default, the tile layer contains a large amount of information that is not needed for a map like this. Such as airports, small city/suburb names, and many street names. Reducing the amount of information shown by the tile layer allows for less distraction to the user.

## Analysis

The statistics created for each county on each date are as follows. The script when processing the data goes through all of the data and takes the cases/deaths per county date record and calculates these statistics which are not initially present in the data.

|Statistic|Method of Creation  |
|--|--|
| Daily increase in cases | Present day's cases - previous day's cases |
| Daily increase in deaths| Present day's deaths - previous day's deaths|
| Cases doubling time (days) | 70 / ( (Average daily case percent increase for past 7 days) * 100 ) |
| Deathsdoubling time (days) | 70 / ( (Average daily deaths percent increase for past 7 days) * 100 ) |
| Number of cases per 100k people | ( cases / county population) * 100,000 |

I will be adding further explanantion of these statistics onto the website in an about page at a later date.


## User Experience

I created this map with user experience in mind. The simple layout aims to be self explanatory and allow anyone to understand what is happening as they interact with it. 

The base layer of the map is the tile layer generated by Mapbox. The thematic layer would be the choropleth with all of the county data in it. The interactive elements consist of the slider at the bottom of the screen for the date, the panel on the left for selecting what statistic to show, and the panel on the right which shows up after a user has clicked on a county.

I made this map a consistent color scheme and tried to keep the effects minimal. There are some rough areas currently with my map, I plan to fix them soon. Here is a list of planned additions to the map:

 - Making the map compatible for mobile devices.
 - Adding a legend to explain what the choropleth colors mean.
 - Ensuring all counties data are being displayed accurately and there are no bugs with this.
 - Building in a timeline view for Washington State which shows some important information and dates.

The application not being responsive is at the top of the priority list as well as doing some testing and getting rid of bugs.

Not being an academically rigorous map, I have chosen to omit things like a north arrow and a scale bar, as I do not think that those are necessary most of the time. 

## Conclusion

The COVID-19 crisis has been highly politicized and has caused massive change in our modern world. The news still focuses on it every day and little is done to inform people on the fact and information beyond the *Flatten the Curve* campaign. The *Flatten the Curve* campaign was highly successful but did not convey the whole message on how a disease spreads geographically. This project aims to lower the informational divide. While it is much easier for a student like me, who has taken many geography and epidemiology classes at a top university, to understand the spread of COVID-19. 

Currently COVID-19 is still having a large impact on our world today. Shops are still closed, barbers are still not cutting hair, and many people are without their stable income. There is reasoning behind the lockdowns which can be hard to see when you do not have all of the information. Once people can understand all of the information related to COVID-19 then they can feel much more confident on having an opinion and trying to shape public policy. Without the facts and information, great harm can be done through ignorance. While my map does not show all of the information surrounding COVID-19, it does give a large amount of information to the user. More importantly, it serves as a one of a kind map giving people access to information that previously was not readily available to them.
