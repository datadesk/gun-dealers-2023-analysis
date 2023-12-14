# gun-dealers-2023-analysis

This analysis is an exploration into the data about gun dealers. Using information about their locations and number we compared trends by state and county and estimated how dealer density effects gun homicides. 

Our findings informed these stories:
- [After mass shooting, Monterey Park limited gun sales. But local laws alone won’t stop violence](https://www.latimes.com/california/story/2023-12-07/gun-dealers-story-1-restrictions)
- [Buying guns for criminals: Easy, illegal and ‘extremely difficult’ to stop](https://www.latimes.com/california/story/2023-12-07/gun-dealers-story-2-straw-purchases)
- [She wanted to open a gun store. They wanted to shut one down. Local laws got in the way](https://www.latimes.com/california/story/2023-12-07/second-gun-dealers-story)

Read more about our process here: _TK: Link to methodology_

_Reporter: Gabrielle LaMarr LeMee (<gabrielle.lamarrlemee@latimes.com>)_

## Dealer trends and dealer density

ATF publishes updated lists of [Federal Firearms Licenses](https://www.atf.gov/firearms/listing-federal-firearms-licensees) on their website each month going back as far as 2014. Through a FOIA request, we received lists from 2003-2013. The `analysis/01-convert-ffl-lists.ipynb` and `analysis/03-clean-ffl-lists.ipynb` notebooks standardize those data. Most importantly for our analysis it filters to just the 50 states and to just the FFL license types that are allowed to deal in firearms.

In the cleaning notebook we also create grouped county-level datasets, merge those with FIPS state and county codes and county area data that is consolidated in `analysis/02-create-county-reference-files.ipynb` to calculate the dealers per 100 square miles figure that will be used in the dealer density effect on gun homicides section.

Using the code in `analysis/11-ffl-trends.ipynb`, we grouped FFLs by year, state, county, and license type to identify differences between places and changes over time.

## Dealer density effect on gun homicides

Following the method laid out in this [2021 academic study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3867782), we conducted a county-level fixed-effects regression comparing dealer density per 100 square miles to counts of gun homicides.  

Through a research request, we obtained non-suppressed county-level mortality data from the [CDC's Wonder database](https://wonder.cdc.gov/Deaths-by-Underlying-Cause.html) for 2010-2021. We used the code in the `analysis/05-clean-cdc-data.ipynb` to filter to deaths from firearm homicide and non-firearm homicide and group by county. Then, using `analysis/09-compile-regression-data.ipynb`, we merge homicide data with the county-level cleaned FFL list data and demographic data on a two year lag.

The code for our regression models is in `analysis/16-density_deaths_poisson_regression.Rmd`.

Some FFLs in ATF's list are not actual gun stores. They may hold licenses to repair guns or use their license mainly as an efficient way to add to their own collections. In order to ensure that using the FFL list is still a good way to detect relative differences between places, we compared dealer density using that list and dealer density using listings on Gunbroker.com. The code for that comparison is in the `analysis/12-gunbroker-analysis.ipynb` notebook.

We analyzed CDC data in the `analysis/15-calculate-homicide-rates.ipynb` notebook to calculate national homicide rates for the past several years.

## Distance to dealers

First we geocode the dealers by running the premise address through Google's geocoding API using the `analysis/04-geocode-ffls.ipynb` notebook.

Next, in the `analysis/07-generate-isochrones.ipynb` notebook, we request isochrones for 10 minute driving distances from each dealer using a local instance of [openrouteservice](https://openrouteservice.org/). We break these requests into states in order to run the maps on our local machine without crashing and then we will piece the state files back together later. California is broken into north and south.

Going through each state, we dissolve the isochrones to get the area statewide that is accessible to at least one dealer in a 10 minute drive using `analysis/08-merge-isochrones.ipynb`. We also use a geopandas union to see where isochrones overlap and group them into bins to create regions that are accessible to larger numbers of dealers within 10 minutes.

Finally, to calculate a count of the population that lives within a 10 minute drive of a dealer, we ran zonal statistics in QGIS using the dissolved regions and NASA's [Gridded Population of the World](https://sedac.ciesin.columbia.edu/data/set/gpw-v4-population-count-rev11) at the 30 arc-second (~1 km at the equator) resolution, a raster file with 2020 Census population estimates. This gave us a result of 302,546,129. Dividing that count with the total popluation, 343,326,439, - higher than the Census estimate, which we arrived at through the same method but using the entire United States – we found that 88% of the population lives within a 10 minute drive of a dealer.

## California crime gun data

California published a [report](https://oag.ca.gov/system/files/attachments/press-docs/AB%201191%20Crime%20Gun%20Report.pdf) in June that contains an analysis of crime guns sold and recovered within the state from 2010 through 2022. At the end are several pages of a table listing each dealer that sold one or more gun that later was recovered by police. Our `analysis/06-clean-ca-crime-gun-data.ipynb` notebook takes the results of a Tabula extract of this table and formats the data. We calculate the average percent of crime gun sold and the top sellers in the `analysis/13-ca-crime-guns-analysis.ipynb` notebook

## ATF crime gun trace data

Data about crime gun traces exists in reports produced by ATF in PDF form and in tables listed on their website. We extracted and compiled those data using the code in the `analysis/10-scrape-atf-trace-data.ipynb` notebook.

In the `analysis/14-atf-trace-analysis.ipynb` we analyzed trace data to determine trends in time to crime, in-state recoveries, and other findings. 
