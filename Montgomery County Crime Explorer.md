Montgomery County Crime Explorer
========================================================
author: Brandon Kopp
date: April 2, 2017
css: custom.css
width: 1440
height: 900

Overview
========================================================
type: exclaim
Understanding the distribution of crime across time and space can help us make more informed decisions.

This application pulls *current* crime reports from an API maintained by the Montgomery County, Maryland government and visualizes them in three ways:
- **Points Map** – Each report is plotted on a map by its latitude and longitude. Clicking on the points provides details about the crime committed.
-  **Heatmap** – Kernel density estimation is used to summarize the co-occurrence of crime within a certain geographic area. The parameters of the heatmap can be tuned using on-screen controls.
- **Plots** – Several charts are provided showing the distribution of crimes across time.

The live application can be viewed on [Shinyapps.io](https://brandonkopp.shinyapps.io/MoCoCrimeExplorer/).  
The code can be viewed on [Github](https://github.com/brandonkopp/MoCo-Crime-Explorer).

Interaction
========================================================
type: exclaim
<br>Users can filter data by **crime type** and **date range**. They can also **add icons** for school, hospital, and liquor store locations.  They also have a full toolbar to **adjust the heatmap**.

<div style="text-align: center"><img src="./Montgomery County Crime Explorer-figure/layout.png"></div>

Getting The Data
========================================================
class: small-code
type: exclaim

The crime reports are downloaded through the [dataMontgomery Crime API](https://data.montgomerycountymd.gov/Public-Safety/Crime/icn6-v9z3) which offers a lot of resources to help developers make use of the data (see also [Socrata](https://www.socrata.com/) ).  

The crime reports are updated daily so users get up-to-date information. Reports in the application refresh when the user enters the application and whenever they change the crime type or date range parameters.

Accessing the data is as simple as passing an [API query](https://dev.socrata.com/foundry/data.montgomerycountymd.gov/icn6-v9z3) and pulling the JSON formatted response.

<h3>API Query</h3>


```r
url <- paste0("https://data.montgomerycountymd.gov/resource/crime.json?start_date%20>%20%27",input$date[1],"%27%20AND%20start_date%20<%20%27",input$date[2])

crime <- fromJSON(url)
```


Kernel Density Estimation
========================================================
class: small-code
type: exclaim
The heatmaps are created using kernel density estimation in the **KernSmooth** package.


```r
library(KernSmooth)
data(geyser, package="MASS")
x <- cbind(geyser$duration, geyser$waiting)
est <- bkde2D(x, bandwidth=c(0.7, 7))
par(mfrow=c(1,2))
contour(est$x1, est$x2, est$fhat)
persp(est$fhat)
```

<img src="Montgomery County Crime Explorer-figure/unnamed-chunk-2-1.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

Leaflet Maps
========================================================
class: small-code
type: exclaim

The maps are the heart of this application. This is the code that was used to create the two types of **Leaflet** base maps.

<h3>Points Map</h3>


```r
leaflet(crime()) %>%
    addProviderTiles("CartoDB.Positron") %>%
    addPolygons(data = moco, weight = 2, color = "black", fillOpacity = 0) %>%
    addCircles(lng = crime()$longitude, lat = crime()$latitude, popup= popup, 
                       weight = 8, radius=8, color= col(crime()$cl), stroke = TRUE, fillOpacity = .6) %>%
    addLegend("bottomright", colors= brewer.pal(nrow(legdf), "Paired"), 
                        labels= legdf$unique.crime......maplabels...,opacity = 0.5)
```

<h3>Heatmap</h3>

```r
leaflet() %>%
    addProviderTiles("CartoDB.Positron") %>%
    addPolygons(data = moco, weight = 2, color = "black", fillOpacity = 0) %>%
    addRasterImage(x = loc_density_raster, colors=color_pal, opacity = input$opacity, project = FALSE)
```
