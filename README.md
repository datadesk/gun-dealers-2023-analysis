# gun-dealers-2023-analysis

This analysis is an exploration into the data about gun dealers. Using information about their locations and number we compared trends by state and county and estimated how dealer density effects gun homicides. 

Our findings informed these stories:
- [After mass shooting, Monterey Park limited gun sales. But local laws alone won’t stop violence](https://www.latimes.com/california/story/2023-12-07/gun-dealers-story-1-restrictions)
- [Buying guns for criminals: Easy, illegal and ‘extremely difficult’ to stop](https://www.latimes.com/california/story/2023-12-07/gun-dealers-story-2-straw-purchases)
- [She wanted to open a gun store. They wanted to shut one down. Local laws got in the way](https://www.latimes.com/california/story/2023-12-07/second-gun-dealers-story)

Read more about our process and here: _TK: Link to methodology_

_Reporter: Gabrielle LaMarr LeMee (<gabrielle.lamarrlemee@latimes.com>)_

## Dealer trends and dealer density

ATF publishes updated lists of [Federal Firearms Licenses](https://www.atf.gov/firearms/listing-federal-firearms-licensees) on their website each month going back as far as 2014. Through a FOIA request, we received lists from 2003-2013. The `analysis/01-clean/clean-ffl-lists.ipynb` notebook standardizes those data. Most importantly for our analysis it filters to just the 50 states and to just the FFL license types that are allowed to deal in firearms. 

In this cleaning notebook we also create grouped county-level datasets, merge those with FIPS state and county codes and county area to calculate the dealers per 100 square miles figure that will be used in the dealer density effect on gun homicides section.

Using the code in `analysis/03-analyze/dealer-trends.ipynb`, we grouped FFLs by year, state, county, and license type to identify differences between places and changes over time.

## Dealer density effect on gun homicides

Following the method laid out in this [2021 academic study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3867782), we conducted a county-level fixed-effects regression comparing dealer density per 100 square miles to counts of gun homicides.  

Through a research request, we obtained non-suppressed county-level mortality data from the [CDC's Wonder database](https://wonder.cdc.gov/Deaths-by-Underlying-Cause.html) for 2010-2021. We used the code in the `analysis/01-clean/clean-cdc-data.ipynb` to filter to deaths from firearm homicide and non-firearm homicide, group by county, and merge with the county-level cleaned FFL list data on a two year lag. 

The code for our regression models is in `analysis/03-analyze/density_deaths_poisson_regression.Rmd`. 

## Distance to dealers

First we geocode the dealers by running the premise address through Google's geocoding API using the `analysis/01-clean/geocode-ffls.ipynb` notebook. 

Next, in the `analysis/02-compile/generate-isochrones.ipynb` notebook, we request isochrones for 10 minute driving distances from each dealer using a local instance of [openrouteservice](https://openrouteservice.org/). We break these requests into states in order to run the maps on our local machine without crashing and then we will piece the state files back together later. California is broken into north and south. 

Going through each state, we dissolve the isochrones to get the area statewide that is accessible to at least one dealer in a 10 minute drive. We also use a geopandas union to see where isochrones overlap and group them into bins to create regions that are accessible to larger numbers of dealers within 10 minutes. 

Finally, to calculate a count of the population that lives within a 10 minute drive of a dealer, we ran zonal statistics in QGIS using the dissolved regions and NASA's [Gridded Population of the World](https://sedac.ciesin.columbia.edu/data/set/gpw-v4-population-count-rev11) at the 30 arc-second (~1 km at the equator) resolution, a raster file with 2020 Census population estimates. This gave us a result of 302,546,129. Dividing that count with the total popluation, 343,326,439, - higher than the Census estimate, which we arrived at through the same method but using the entire United States – we found that 88% of the population lives within a 10 minute drive of a dealer. 

## Crime gun data

California published a [report](https://oag.ca.gov/system/files/attachments/press-docs/AB%201191%20Crime%20Gun%20Report.pdf) in June that contains an analysis of crime guns sold and recovered within the state from 2010 through 2022. At the end are several pages of a table listing each dealer that sold one or more gun that later was recovered by police. Our `analysis/01-clean/clean-ca-crime-gun-data.ipynb` notebook takes the results of a Tabula extract of this table and formats the data. We calculate the average percent of crime gun sold and the top sellers in the `analysis/03-analyze/ca-crime-guns.ipynb` notebook

_TK: ATF trace data analysis_

## Project setup instructions

After cloning the git repo:

`datakit data pull` to retrieve the data files.
