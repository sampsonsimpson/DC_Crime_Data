DC Crime Map - Pitch - Developing Data Products Week 4
========================================================
author: Neil Kutty
date: June 5, 2017
autosize: true

Why is a DC Crime Map Useful?
========================================================
### The Usefulness of Summary Stats with a Map View.
<br>

  + This map allows for the user to not only see clusters of recent crimes in the city, but they also get updatable set of summary statistics for the viewable area of the map. 

  + The user gets access to the underlying table for all map points & charts.  

  + This 



Getting and Cleaning the dataset
========================================================


```r
library(dplyr)
library(tidyr)
library(jsonlite)
library(lubridate)
library(leaflet)
# Fig. 1 

########---------------------------------------------------------------------#>>>
  ## Retrieve the data in JSON format from opendata.dc.gov using fromJson()
  dccrimejsonlite <- fromJSON('http://opendata.dc.gov/datasets/dc3289eab3d2400ea49c154863312434_8.geojson')
  ## use cbind() combine the list elements and create a dataframe
  dc_crime_json <- cbind(dccrimejsonlite$features$properties,dccrimejsonlite$features$geometry)

  ## Seperate and clean lat/long columns but keep original datetime column
  ## --also separate REPORTDATETIME column
  dc_crime_clean <- dc_crime_json %>% 
    separate(coordinates, into = c("X", "Y"), sep = ",")%>%
    separate(REPORT_DAT, into = c("Date","Time"), sep="T", remove = FALSE)%>%
    mutate(Weekday = weekdays(as.Date(REPORT_DAT)),
           DATETIME = ymd_hms(REPORT_DAT, tz='America/New_York'),
           Date = as.Date(Date),
           X = as.numeric(gsub("c\\(","",X)),
           Y = as.numeric(gsub("\\)","",Y)))
```

Rendered Leaflet Map
===
<font size='5'>

```r
points <- cbind(dc_crime_clean$X,dc_crime_clean$Y)
leaflet() %>%
  addProviderTiles("OpenStreetMap.Mapnik",
                   options = providerTileOptions(noWrap = TRUE)
  ) %>%
  addMarkers(data = points,
             popup = paste0("<strong>Report Date: </strong>",
                            dc_crime_clean$DateClean,
                            "<br><strong>Offense: </strong>", 
                            dc_crime_clean$OFFENSE, 
                            "<br><strong>method: </strong>", 
                            dc_crime_clean$METHOD,
                            "<br><strong>shift: </strong>",
                            dc_crime_clean$SHIFT,
                            "<br><strong>blocksite address: </strong><br>",
                            dc_crime_clean$BLOCKSITEADDRESS
             ),
             clusterOptions = markerClusterOptions()
  ) 
```
</font>
***
![plot of chunk mapRender](DCCrimeMapPitch-figure/mapRender-1.png)