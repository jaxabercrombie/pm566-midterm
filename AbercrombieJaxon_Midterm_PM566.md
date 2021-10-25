## Introduction

The ongoing COVID-19 pandemic has come under greater control with the
roll-out of COVID-19 vaccines starting back in December 2020.
California’s governor, Gavin Newsom, speaks regularly about the success
that our state has had with controlling the virus through immunization
efforts. While California is certainly diverse and varies in demographic
composition and physical environment by county, investigating how
vaccine uptake has varied by county would be fascinating. Identifying
which counties are behind on COVID-19 immunizations would be crucial for
intervening and attempting to increase the percent of vaccinated
individuals in a more targeted fashion.

Furthermore, toward the beginning of the vaccine roll-out, discourse
about which vaccine company people should receive their dose(s) from was
frequent. Whether someone received Moderna, Pfizer, or Johnson & Johnson
vaccines would dictate whether they received one or two doses and had
greater immunity against variants. Because of this, investigating how
California counties may differ by vaccine company distribution would
also be exciting. Additionally, comparing vaccination rates alongside
cases and deaths may reveal changes in vaccination attitudes based on
the ebb and flow of COVID-19 morbidity and mortality. For example, if
cases for a specific variant grew largely over summer, would it be
expected to have more vaccine uptake out of fear?

All that said, the primary question at hand is: *How have COVID-19
vaccination rates varied by county in California since their initial
roll-out?* Furthermore, there are two secondary questions that dig
deeper into the data sets used: (1) *How do vaccination efforts vary by
vaccine company across these counties (Pfizer, Moderna, Johnson &
Johnson)?* and (2) *How do trends in cases and deaths potentially affect
immunization rates for California as a whole?*

## Methods

There were two different sets of data used for this project: one
including data for CA counties and their administered vaccine doses and
another with data regarding COVID-19 cases and deaths for each CA
county. The data about vaccine doses for each county, titled “Vaccines
by California County,” came from LA city’s data site at [this
link](https://data.lacity.org/COVID-19/Vaccines-by-California-County/rpp7-mevy),
and it was downloaded by a CSV file. This particular set has data from
the start of vaccine roll-out in mid-December 2020 up until mid-October
2021 and includes dose data for each vaccine company per county, county
population count, and administration date. Data regarding COVID-related
cases and deaths came from the California Department of Health and Human
Services at [this
link](https://data.chhs.ca.gov/dataset/covid-19-time-series-metrics-by-county-and-state/resource/046cdd2b-31e5-4d34-9ed3-b48cdbc4be7a),
and it was also downloaded as a CSV file. COVID-related cases and
deaths, both raw and cumulative, are included for every day since
February 2020, and the number of total tests conducted is also included.

#### Data Wrangling

The data was first read and stored in two different variables.

    vaxCA <- read.csv("CA Vaccine Data.csv")
    covidCA <- read.csv("CA Deaths and Cases.csv")

Because the sets would be combined by county and date, ensuring
consistency in date format and variable name for county was important.
After ensuring identical format and variable names, the two data sets
were combined. Then, to filter the combined set by when vaccine roll-out
began, the date set was altered to contain only dates from December 15,
2020 onward. Additionally, the date variable was altered to not include
dates past October 20, 2021 because there was missing data for more
recent dates.

Upon browsing the merged data set, it appeared that many rows existed
for each county for the dates of August 30, 2021 onward. To fix this,
the average was applied for variables for those dates, essentially
combining multiple instances into one for the duplicate date-county
combinations.

    covidCA <-
      covidCA %>%
      mutate(county = area)

    vaxCA <-
      vaxCA %>%
      mutate(date = as.Date(vaxCA$date, "%m/%d/%Y"))

    mergedCA <- merge(vaxCA, covidCA, 
                      by = c("date","county"),
                      all.x = TRUE)

    mergedCA <-
      mergedCA %>%
      filter(date >= as.Date("2020-12-15")) %>%
      filter(date <= as.Date("2021-10-20")) %>%
      group_by(date, county) %>% 
      summarise(across(c(total_doses,cumulative_total_doses,
                         pfizer_doses,cumulative_pfizer_doses,
                         moderna_doses,cumulative_moderna_doses,
                         jj_doses,cumulative_jj_doses,
                         partially_vaccinated,
                         total_partially_vaccinated,fully_vaccinated,
                         cumulative_fully_vaccinated,at_least_one_dose,
                         cumulative_at_least_one_dose,population,
                         cases,cumulative_cases,deaths,
                         cumulative_deaths,total_tests,
                         cumulative_total_tests,positive_tests,
                         cumulative_positive_tests), mean, .groups = date))

Additionally, variables were created for later ease with analysis and
normalize county data to make it comparable to others. The variables
created were transformations of the main variables to be used in
analysis:

1.  *dose\_standard*: the number of cumulative total doses over time
    standardized by population size to make comparisons

2.  *perc\_vaccinated / perc\_partial*: the percent of a county’s
    population that is fully vaccinated / partially vaccinated

<!-- -->

    mergedCA <-
      mergedCA %>%
      mutate(dose_standard = (cumulative_total_doses/population),
             perc_vaccinated = (cumulative_fully_vaccinated/population)*100,
             perc_partial = (cumulative_at_least_one_dose/population)*100)

To ensure missing values would not affect our analysis, the number of
NAs were summed.

    mergedNA <- sum(is.na(mergedCA))

Fortunately, there were 0 NA values after cleaning and wrangling data.

#### Exploratory Data Analysis

Now that the *mergedCA* data is cleaned and wrangled, it can be explored
more. A check on the expected number of observations was performed.

    expObs <- length(unique(mergedCA$date))*length(unique(mergedCA$county))
    actualObs <- nrow(mergedCA)

The number of expected observations after wrangling matches the observed
number of observations (exp:17980, obs: 17980).

Following this check, summary statistics using the “knitr” package were
produced and displayed in the preliminary results section before data
visualization and further analysis. Three summary tables were created:
one for population data (population count, cases, deaths), one for
vaccine status data, and one for vaccine company data.

#### Data Visualization

The main tool used to create visualizations of data was *ggplot2*. The
package offered ways to create appealing bar graphs, time series plots,
and use *facet\_wrap() t*o de-clutter plots and focus on data
county-by-county.

The visualizations created were:

1.  Figure 1: A time series plot depicting how vaccination rates have
    changed over time by county

2.  Figure 2: A bar graph depicting counties by highest percent fully
    vaccinated, in descending order

3.  Figure 3: Pie charts demonstrating the distribution of different
    vaccine company dose administrations by county

4.  Figure 4: A vertically-aligned grid of time series plots (one of
    total cumulative vaccine doses, one of cases, and one of deaths) to
    see if trends in mortality/morbidity are related to
    increased/decreased vaccine hesitancy

## Preliminary Results

For some results involving vaccine company differences, it should be
acknowledged that Johnson & Johnson requires one dose to be considered
fully vaccinated, while it takes two for Moderna and Pfizer. That
reality alone may affect statistics involving Johnson & Johnson dose
percentages because only one dose would be taken compared to two for
other companies’ vaccines.

#### Summary Tables

Three summary tables for the data were created to assess minimums,
maximums, averages, and standard deviations to ensure no variables were
worrisome before continuing with analysis. Variables checked through
these tables were those that had not undergone transformation (ex:
*fully\_vaccinated* rather than *perc\_vaccinated* or
*cumulative\_fully\_vaccinated*) to avoid redundancy.

The first table includes data regarding vaccine dose counts for each
county, and it also is stratified by vaccine company.

    mergedCA %>% 
      group_by(county) %>%
      summarise(Cases_min = min(cases), 
                Cases_mean = mean(cases),
                Cases_max = max(cases),
                Cases_sd = sd(cases),
                Deaths_min = min(deaths), 
                Deaths_mean = mean(deaths),
                Deaths_max = max(deaths),
                Deaths_sd = sd(deaths),
                Pop_min = min(population), 
                Pop_mean = mean(population),
                Pop_max = max(population),
                Pop_sd = sd(population)) %>% 
      knitr::kable(col.names = c("County", 
                                 "Min Cases", 
                                 "Mean Cases", 
                                 "Max Cases",
                                 "SD Cases",
                                 "Min Deaths", 
                                 "Mean Deaths", 
                                 "Max Deaths",
                                 "SD Deaths",
                                 "Min Pop", 
                                 "Mean Pop", 
                                 "Max Pop",
                                 "SD Pop"), digits = 2, "pipe")

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 7%" />
<col style="width: 8%" />
<col style="width: 7%" />
<col style="width: 6%" />
<col style="width: 8%" />
<col style="width: 8%" />
<col style="width: 8%" />
<col style="width: 7%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 5%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">County</th>
<th style="text-align: right;">Min Cases</th>
<th style="text-align: right;">Mean Cases</th>
<th style="text-align: right;">Max Cases</th>
<th style="text-align: right;">SD Cases</th>
<th style="text-align: right;">Min Deaths</th>
<th style="text-align: right;">Mean Deaths</th>
<th style="text-align: right;">Max Deaths</th>
<th style="text-align: right;">SD Deaths</th>
<th style="text-align: right;">Min Pop</th>
<th style="text-align: right;">Mean Pop</th>
<th style="text-align: right;">Max Pop</th>
<th style="text-align: right;">SD Pop</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Alameda</td>
<td style="text-align: right;">14</td>
<td style="text-align: right;">240.75</td>
<td style="text-align: right;">1242</td>
<td style="text-align: right;">245.10</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.86</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">3.85</td>
<td style="text-align: right;">1685886</td>
<td style="text-align: right;">1685886</td>
<td style="text-align: right;">1685886</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Alpine</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.13</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.43</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">1117</td>
<td style="text-align: right;">1117</td>
<td style="text-align: right;">1117</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Amador</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.50</td>
<td style="text-align: right;">121</td>
<td style="text-align: right;">15.54</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.14</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.40</td>
<td style="text-align: right;">38531</td>
<td style="text-align: right;">38531</td>
<td style="text-align: right;">38531</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Butte</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">45.23</td>
<td style="text-align: right;">191</td>
<td style="text-align: right;">45.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.57</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.92</td>
<td style="text-align: right;">217769</td>
<td style="text-align: right;">217769</td>
<td style="text-align: right;">217769</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Calaveras</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">9.71</td>
<td style="text-align: right;">61</td>
<td style="text-align: right;">11.37</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.18</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.45</td>
<td style="text-align: right;">44289</td>
<td style="text-align: right;">44289</td>
<td style="text-align: right;">44289</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Colusa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.36</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">5.83</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.03</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.18</td>
<td style="text-align: right;">22593</td>
<td style="text-align: right;">22593</td>
<td style="text-align: right;">22593</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Contra Costa</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">206.05</td>
<td style="text-align: right;">931</td>
<td style="text-align: right;">196.51</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.22</td>
<td style="text-align: right;">15</td>
<td style="text-align: right;">2.82</td>
<td style="text-align: right;">1160099</td>
<td style="text-align: right;">1160099</td>
<td style="text-align: right;">1160099</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Del Norte</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">9.35</td>
<td style="text-align: right;">72</td>
<td style="text-align: right;">13.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.12</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.46</td>
<td style="text-align: right;">27558</td>
<td style="text-align: right;">27558</td>
<td style="text-align: right;">27558</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">El Dorado</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">34.51</td>
<td style="text-align: right;">213</td>
<td style="text-align: right;">35.04</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.42</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.89</td>
<td style="text-align: right;">193098</td>
<td style="text-align: right;">193098</td>
<td style="text-align: right;">193098</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Fresno</td>
<td style="text-align: right;">9</td>
<td style="text-align: right;">248.90</td>
<td style="text-align: right;">1369</td>
<td style="text-align: right;">278.33</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.45</td>
<td style="text-align: right;">23</td>
<td style="text-align: right;">5.27</td>
<td style="text-align: right;">1032227</td>
<td style="text-align: right;">1032227</td>
<td style="text-align: right;">1032227</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Glenn</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">6.77</td>
<td style="text-align: right;">35</td>
<td style="text-align: right;">7.92</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.06</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.25</td>
<td style="text-align: right;">29348</td>
<td style="text-align: right;">29348</td>
<td style="text-align: right;">29348</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Humboldt</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">23.65</td>
<td style="text-align: right;">91</td>
<td style="text-align: right;">19.26</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.20</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.48</td>
<td style="text-align: right;">134098</td>
<td style="text-align: right;">134098</td>
<td style="text-align: right;">134098</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Imperial</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">38.58</td>
<td style="text-align: right;">285</td>
<td style="text-align: right;">48.63</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.09</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">2.16</td>
<td style="text-align: right;">191649</td>
<td style="text-align: right;">191649</td>
<td style="text-align: right;">191649</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Inyo</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5.19</td>
<td style="text-align: right;">32</td>
<td style="text-align: right;">6.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.07</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.26</td>
<td style="text-align: right;">18453</td>
<td style="text-align: right;">18453</td>
<td style="text-align: right;">18453</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Kern</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">216.49</td>
<td style="text-align: right;">1271</td>
<td style="text-align: right;">257.99</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.45</td>
<td style="text-align: right;">18</td>
<td style="text-align: right;">4.03</td>
<td style="text-align: right;">927251</td>
<td style="text-align: right;">927251</td>
<td style="text-align: right;">927251</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kings</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">53.08</td>
<td style="text-align: right;">271</td>
<td style="text-align: right;">57.20</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.70</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">1.07</td>
<td style="text-align: right;">156444</td>
<td style="text-align: right;">156444</td>
<td style="text-align: right;">156444</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lake</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16.13</td>
<td style="text-align: right;">79</td>
<td style="text-align: right;">15.59</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.25</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">0.56</td>
<td style="text-align: right;">64871</td>
<td style="text-align: right;">64871</td>
<td style="text-align: right;">64871</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lassen</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">9.85</td>
<td style="text-align: right;">172</td>
<td style="text-align: right;">19.45</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.06</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.28</td>
<td style="text-align: right;">30065</td>
<td style="text-align: right;">30065</td>
<td style="text-align: right;">30065</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Los Angeles</td>
<td style="text-align: right;">48</td>
<td style="text-align: right;">2662.55</td>
<td style="text-align: right;">22267</td>
<td style="text-align: right;">4386.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">56.02</td>
<td style="text-align: right;">305</td>
<td style="text-align: right;">82.81</td>
<td style="text-align: right;">10257557</td>
<td style="text-align: right;">10257557</td>
<td style="text-align: right;">10257557</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Madera</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">42.57</td>
<td style="text-align: right;">314</td>
<td style="text-align: right;">49.15</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.48</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.86</td>
<td style="text-align: right;">160089</td>
<td style="text-align: right;">160089</td>
<td style="text-align: right;">160089</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Marin</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">26.92</td>
<td style="text-align: right;">152</td>
<td style="text-align: right;">27.66</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.32</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.68</td>
<td style="text-align: right;">260800</td>
<td style="text-align: right;">260800</td>
<td style="text-align: right;">260800</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mariposa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.20</td>
<td style="text-align: right;">30</td>
<td style="text-align: right;">4.47</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.01</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.08</td>
<td style="text-align: right;">17795</td>
<td style="text-align: right;">17795</td>
<td style="text-align: right;">17795</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mendocino</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16.58</td>
<td style="text-align: right;">69</td>
<td style="text-align: right;">15.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.19</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.49</td>
<td style="text-align: right;">88439</td>
<td style="text-align: right;">88439</td>
<td style="text-align: right;">88439</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Merced</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">79.95</td>
<td style="text-align: right;">470</td>
<td style="text-align: right;">85.18</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.17</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">1.39</td>
<td style="text-align: right;">287420</td>
<td style="text-align: right;">287420</td>
<td style="text-align: right;">287420</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Modoc</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.98</td>
<td style="text-align: right;">16</td>
<td style="text-align: right;">1.86</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.15</td>
<td style="text-align: right;">9475</td>
<td style="text-align: right;">9475</td>
<td style="text-align: right;">9475</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mono</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.89</td>
<td style="text-align: right;">31</td>
<td style="text-align: right;">4.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">13961</td>
<td style="text-align: right;">13961</td>
<td style="text-align: right;">13961</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Monterey</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">80.94</td>
<td style="text-align: right;">833</td>
<td style="text-align: right;">145.72</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.31</td>
<td style="text-align: right;">14</td>
<td style="text-align: right;">2.43</td>
<td style="text-align: right;">448732</td>
<td style="text-align: right;">448732</td>
<td style="text-align: right;">448732</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Napa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">25.67</td>
<td style="text-align: right;">141</td>
<td style="text-align: right;">27.73</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.24</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">0.58</td>
<td style="text-align: right;">139652</td>
<td style="text-align: right;">139652</td>
<td style="text-align: right;">139652</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Nevada</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">21.07</td>
<td style="text-align: right;">87</td>
<td style="text-align: right;">19.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.16</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.50</td>
<td style="text-align: right;">98710</td>
<td style="text-align: right;">98710</td>
<td style="text-align: right;">98710</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Orange</td>
<td style="text-align: right;">18</td>
<td style="text-align: right;">575.54</td>
<td style="text-align: right;">4385</td>
<td style="text-align: right;">889.23</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.60</td>
<td style="text-align: right;">69</td>
<td style="text-align: right;">17.71</td>
<td style="text-align: right;">3228519</td>
<td style="text-align: right;">3228519</td>
<td style="text-align: right;">3228519</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Placer</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">77.87</td>
<td style="text-align: right;">341</td>
<td style="text-align: right;">68.79</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.99</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">1.43</td>
<td style="text-align: right;">400434</td>
<td style="text-align: right;">400434</td>
<td style="text-align: right;">400434</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Plumas</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.99</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">5.52</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.13</td>
<td style="text-align: right;">18997</td>
<td style="text-align: right;">18997</td>
<td style="text-align: right;">18997</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Riverside</td>
<td style="text-align: right;">20</td>
<td style="text-align: right;">684.69</td>
<td style="text-align: right;">4839</td>
<td style="text-align: right;">1034.79</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.18</td>
<td style="text-align: right;">59</td>
<td style="text-align: right;">14.85</td>
<td style="text-align: right;">2468145</td>
<td style="text-align: right;">2468145</td>
<td style="text-align: right;">2468145</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sacramento</td>
<td style="text-align: right;">22</td>
<td style="text-align: right;">322.87</td>
<td style="text-align: right;">1228</td>
<td style="text-align: right;">268.13</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.48</td>
<td style="text-align: right;">19</td>
<td style="text-align: right;">4.29</td>
<td style="text-align: right;">1567975</td>
<td style="text-align: right;">1567975</td>
<td style="text-align: right;">1567975</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Benito</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">14.42</td>
<td style="text-align: right;">119</td>
<td style="text-align: right;">21.33</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.15</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.41</td>
<td style="text-align: right;">64022</td>
<td style="text-align: right;">64022</td>
<td style="text-align: right;">64022</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Bernardino</td>
<td style="text-align: right;">23</td>
<td style="text-align: right;">618.63</td>
<td style="text-align: right;">5284</td>
<td style="text-align: right;">1035.73</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.97</td>
<td style="text-align: right;">71</td>
<td style="text-align: right;">18.17</td>
<td style="text-align: right;">2217398</td>
<td style="text-align: right;">2217398</td>
<td style="text-align: right;">2217398</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Diego</td>
<td style="text-align: right;">22</td>
<td style="text-align: right;">782.72</td>
<td style="text-align: right;">5255</td>
<td style="text-align: right;">972.35</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">9.32</td>
<td style="text-align: right;">58</td>
<td style="text-align: right;">13.13</td>
<td style="text-align: right;">3370418</td>
<td style="text-align: right;">3370418</td>
<td style="text-align: right;">3370418</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Francisco</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">101.48</td>
<td style="text-align: right;">498</td>
<td style="text-align: right;">97.85</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.48</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">2.21</td>
<td style="text-align: right;">892280</td>
<td style="text-align: right;">892280</td>
<td style="text-align: right;">892280</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Joaquin</td>
<td style="text-align: right;">16</td>
<td style="text-align: right;">192.60</td>
<td style="text-align: right;">1101</td>
<td style="text-align: right;">206.34</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.45</td>
<td style="text-align: right;">20</td>
<td style="text-align: right;">4.03</td>
<td style="text-align: right;">782545</td>
<td style="text-align: right;">782545</td>
<td style="text-align: right;">782545</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Luis Obispo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">64.32</td>
<td style="text-align: right;">486</td>
<td style="text-align: right;">81.80</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.86</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">1.40</td>
<td style="text-align: right;">278862</td>
<td style="text-align: right;">278862</td>
<td style="text-align: right;">278862</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Mateo</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">102.26</td>
<td style="text-align: right;">592</td>
<td style="text-align: right;">119.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.14</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">2.23</td>
<td style="text-align: right;">778001</td>
<td style="text-align: right;">778001</td>
<td style="text-align: right;">778001</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Barbara</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">92.05</td>
<td style="text-align: right;">717</td>
<td style="text-align: right;">114.50</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.21</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">2.03</td>
<td style="text-align: right;">456373</td>
<td style="text-align: right;">456373</td>
<td style="text-align: right;">456373</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Santa Clara</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">284.55</td>
<td style="text-align: right;">1754</td>
<td style="text-align: right;">353.45</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.27</td>
<td style="text-align: right;">33</td>
<td style="text-align: right;">6.65</td>
<td style="text-align: right;">1967585</td>
<td style="text-align: right;">1967585</td>
<td style="text-align: right;">1967585</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Cruz</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">40.98</td>
<td style="text-align: right;">300</td>
<td style="text-align: right;">57.13</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.47</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">1.02</td>
<td style="text-align: right;">273999</td>
<td style="text-align: right;">273999</td>
<td style="text-align: right;">273999</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Shasta</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">42.27</td>
<td style="text-align: right;">178</td>
<td style="text-align: right;">39.80</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.78</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">1.16</td>
<td style="text-align: right;">177925</td>
<td style="text-align: right;">177925</td>
<td style="text-align: right;">177925</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sierra</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.47</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">1.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">3115</td>
<td style="text-align: right;">3115</td>
<td style="text-align: right;">3115</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Siskiyou</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">7.52</td>
<td style="text-align: right;">45</td>
<td style="text-align: right;">7.43</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.13</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.40</td>
<td style="text-align: right;">43956</td>
<td style="text-align: right;">43956</td>
<td style="text-align: right;">43956</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Solano</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">95.37</td>
<td style="text-align: right;">574</td>
<td style="text-align: right;">104.37</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.70</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">1.13</td>
<td style="text-align: right;">444255</td>
<td style="text-align: right;">444255</td>
<td style="text-align: right;">444255</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sonoma</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">77.41</td>
<td style="text-align: right;">390</td>
<td style="text-align: right;">76.65</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.75</td>
<td style="text-align: right;">6</td>
<td style="text-align: right;">1.19</td>
<td style="text-align: right;">496668</td>
<td style="text-align: right;">496668</td>
<td style="text-align: right;">496668</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Stanislaus</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">151.91</td>
<td style="text-align: right;">687</td>
<td style="text-align: right;">135.36</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.93</td>
<td style="text-align: right;">11</td>
<td style="text-align: right;">2.33</td>
<td style="text-align: right;">562303</td>
<td style="text-align: right;">562303</td>
<td style="text-align: right;">562303</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sutter</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">26.13</td>
<td style="text-align: right;">110</td>
<td style="text-align: right;">25.24</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.42</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.81</td>
<td style="text-align: right;">105747</td>
<td style="text-align: right;">105747</td>
<td style="text-align: right;">105747</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tehama</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">17.97</td>
<td style="text-align: right;">87</td>
<td style="text-align: right;">19.26</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.25</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.53</td>
<td style="text-align: right;">65885</td>
<td style="text-align: right;">65885</td>
<td style="text-align: right;">65885</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Trinity</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.35</td>
<td style="text-align: right;">14</td>
<td style="text-align: right;">2.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.01</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.10</td>
<td style="text-align: right;">13354</td>
<td style="text-align: right;">13354</td>
<td style="text-align: right;">13354</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tulare</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">118.89</td>
<td style="text-align: right;">682</td>
<td style="text-align: right;">140.96</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.86</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">2.70</td>
<td style="text-align: right;">484423</td>
<td style="text-align: right;">484423</td>
<td style="text-align: right;">484423</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tuolumne</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">15.28</td>
<td style="text-align: right;">311</td>
<td style="text-align: right;">23.52</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.20</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.51</td>
<td style="text-align: right;">52351</td>
<td style="text-align: right;">52351</td>
<td style="text-align: right;">52351</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ventura</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">211.05</td>
<td style="text-align: right;">1824</td>
<td style="text-align: right;">320.80</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.97</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">4.47</td>
<td style="text-align: right;">852747</td>
<td style="text-align: right;">852747</td>
<td style="text-align: right;">852747</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Yolo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">39.02</td>
<td style="text-align: right;">214</td>
<td style="text-align: right;">39.81</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.42</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.85</td>
<td style="text-align: right;">223612</td>
<td style="text-align: right;">223612</td>
<td style="text-align: right;">223612</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: left;">Yuba</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">21.62</td>
<td style="text-align: right;">94</td>
<td style="text-align: right;">20.21</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.15</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.43</td>
<td style="text-align: right;">79290</td>
<td style="text-align: right;">79290</td>
<td style="text-align: right;">79290</td>
<td style="text-align: right;">0</td>
</tr>
</tbody>
</table>

Based on the above table, it is evident that more populous counties
experience more cases and deaths overall, which is expected and likely
relies on population density to happen. Because this data has not yet
spanned an entire year, population counts have not updated in the data.
Essentially, the population count has remained the same for data despite
time elapsing, people being born, and people passing away. The minimum
values of the chosen variables do not reach into negative values, and
the maximum values are not unusually high, which is good. Data in this
table was validated by an [external
source](https://usafacts.org/visualizations/coronavirus-covid-19-spread-map/state/california)
by comparing some counties’ maximum cases and deaths to the source’s
dashboard.

The second table includes data about vaccination status by county, split
up by whether individuals were partially or fully vaccinated.

    mergedCA %>% 
      group_by(county) %>%
      summarise(Partially_min = min(partially_vaccinated), 
                Partially_mean = mean(partially_vaccinated),
                Partially_max = max(partially_vaccinated),
                Partially_sd = sd(partially_vaccinated),
                Fully_min = min(fully_vaccinated), 
                Fully_mean = mean(fully_vaccinated),
                Fully_max = max(fully_vaccinated),
                Fully_sd = sd(fully_vaccinated),) %>% 
      knitr::kable(col.names = c("County", 
                                 "Min Partial Vax", 
                                 "Mean Partial Vax", 
                                 "Max Partial Vax",
                                 "SD Partial Vax",
                                 "Min Full Vax", 
                                 "Mean Full Vax", 
                                 "Max Full Vax",
                                 "SD Full Vax"), digits = 2, "pipe")

<table>
<colgroup>
<col style="width: 12%" />
<col style="width: 12%" />
<col style="width: 12%" />
<col style="width: 12%" />
<col style="width: 11%" />
<col style="width: 9%" />
<col style="width: 10%" />
<col style="width: 9%" />
<col style="width: 9%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">County</th>
<th style="text-align: right;">Min Partial Vax</th>
<th style="text-align: right;">Mean Partial Vax</th>
<th style="text-align: right;">Max Partial Vax</th>
<th style="text-align: right;">SD Partial Vax</th>
<th style="text-align: right;">Min Full Vax</th>
<th style="text-align: right;">Mean Full Vax</th>
<th style="text-align: right;">Max Full Vax</th>
<th style="text-align: right;">SD Full Vax</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Alameda</td>
<td style="text-align: right;">41</td>
<td style="text-align: right;">3708.75</td>
<td style="text-align: right;">17155</td>
<td style="text-align: right;">3667.65</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3778.43</td>
<td style="text-align: right;">15691</td>
<td style="text-align: right;">3945.11</td>
</tr>
<tr class="even">
<td style="text-align: left;">Alpine</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.72</td>
<td style="text-align: right;">72</td>
<td style="text-align: right;">7.98</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.23</td>
<td style="text-align: right;">86</td>
<td style="text-align: right;">8.04</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Amador</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">70.00</td>
<td style="text-align: right;">951</td>
<td style="text-align: right;">104.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">55.85</td>
<td style="text-align: right;">470</td>
<td style="text-align: right;">82.65</td>
</tr>
<tr class="even">
<td style="text-align: left;">Butte</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">334.70</td>
<td style="text-align: right;">2420</td>
<td style="text-align: right;">411.74</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">325.81</td>
<td style="text-align: right;">2617</td>
<td style="text-align: right;">416.36</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Calaveras</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">75.92</td>
<td style="text-align: right;">619</td>
<td style="text-align: right;">105.69</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">67.12</td>
<td style="text-align: right;">589</td>
<td style="text-align: right;">100.60</td>
</tr>
<tr class="even">
<td style="text-align: left;">Colusa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">35.31</td>
<td style="text-align: right;">336</td>
<td style="text-align: right;">48.25</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">33.69</td>
<td style="text-align: right;">308</td>
<td style="text-align: right;">48.40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Contra Costa</td>
<td style="text-align: right;">20</td>
<td style="text-align: right;">2660.88</td>
<td style="text-align: right;">12144</td>
<td style="text-align: right;">2695.96</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2659.40</td>
<td style="text-align: right;">12122</td>
<td style="text-align: right;">2839.15</td>
</tr>
<tr class="even">
<td style="text-align: left;">Del Norte</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">37.26</td>
<td style="text-align: right;">310</td>
<td style="text-align: right;">51.21</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">36.28</td>
<td style="text-align: right;">302</td>
<td style="text-align: right;">50.47</td>
</tr>
<tr class="odd">
<td style="text-align: left;">El Dorado</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">342.61</td>
<td style="text-align: right;">2013</td>
<td style="text-align: right;">340.11</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">333.65</td>
<td style="text-align: right;">1761</td>
<td style="text-align: right;">349.83</td>
</tr>
<tr class="even">
<td style="text-align: left;">Fresno</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">1731.72</td>
<td style="text-align: right;">9066</td>
<td style="text-align: right;">1641.78</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1622.02</td>
<td style="text-align: right;">8196</td>
<td style="text-align: right;">1668.54</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Glenn</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">42.76</td>
<td style="text-align: right;">416</td>
<td style="text-align: right;">63.82</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">43.17</td>
<td style="text-align: right;">511</td>
<td style="text-align: right;">69.59</td>
</tr>
<tr class="even">
<td style="text-align: left;">Humboldt</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">250.19</td>
<td style="text-align: right;">1594</td>
<td style="text-align: right;">300.72</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">249.40</td>
<td style="text-align: right;">1872</td>
<td style="text-align: right;">314.99</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Imperial</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">427.25</td>
<td style="text-align: right;">3190</td>
<td style="text-align: right;">450.68</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">393.46</td>
<td style="text-align: right;">2733</td>
<td style="text-align: right;">444.43</td>
</tr>
<tr class="even">
<td style="text-align: left;">Inyo</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">30.81</td>
<td style="text-align: right;">504</td>
<td style="text-align: right;">53.04</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">29.70</td>
<td style="text-align: right;">505</td>
<td style="text-align: right;">52.06</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Kern</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1333.04</td>
<td style="text-align: right;">4657</td>
<td style="text-align: right;">1062.78</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1261.79</td>
<td style="text-align: right;">5992</td>
<td style="text-align: right;">1125.78</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kings</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">189.38</td>
<td style="text-align: right;">1573</td>
<td style="text-align: right;">191.59</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">177.59</td>
<td style="text-align: right;">1258</td>
<td style="text-align: right;">194.04</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lake</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">103.79</td>
<td style="text-align: right;">575</td>
<td style="text-align: right;">120.72</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">99.98</td>
<td style="text-align: right;">872</td>
<td style="text-align: right;">142.08</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lassen</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">22.46</td>
<td style="text-align: right;">1057</td>
<td style="text-align: right;">64.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">23.46</td>
<td style="text-align: right;">921</td>
<td style="text-align: right;">57.63</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Los Angeles</td>
<td style="text-align: right;">373</td>
<td style="text-align: right;">20846.27</td>
<td style="text-align: right;">74576</td>
<td style="text-align: right;">18108.62</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">19970.44</td>
<td style="text-align: right;">76752</td>
<td style="text-align: right;">18681.70</td>
</tr>
<tr class="even">
<td style="text-align: left;">Madera</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">239.18</td>
<td style="text-align: right;">1123</td>
<td style="text-align: right;">231.44</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">225.38</td>
<td style="text-align: right;">1184</td>
<td style="text-align: right;">240.04</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Marin</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">650.15</td>
<td style="text-align: right;">3301</td>
<td style="text-align: right;">756.46</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">644.96</td>
<td style="text-align: right;">3363</td>
<td style="text-align: right;">778.23</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mariposa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">27.12</td>
<td style="text-align: right;">372</td>
<td style="text-align: right;">52.26</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">19.44</td>
<td style="text-align: right;">397</td>
<td style="text-align: right;">42.39</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mendocino</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">176.58</td>
<td style="text-align: right;">1878</td>
<td style="text-align: right;">268.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">165.98</td>
<td style="text-align: right;">1859</td>
<td style="text-align: right;">263.54</td>
</tr>
<tr class="even">
<td style="text-align: left;">Merced</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">433.54</td>
<td style="text-align: right;">2351</td>
<td style="text-align: right;">417.51</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">357.70</td>
<td style="text-align: right;">2026</td>
<td style="text-align: right;">364.14</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Modoc</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.35</td>
<td style="text-align: right;">173</td>
<td style="text-align: right;">24.75</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.87</td>
<td style="text-align: right;">186</td>
<td style="text-align: right;">26.97</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mono</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">29.18</td>
<td style="text-align: right;">819</td>
<td style="text-align: right;">89.39</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">27.69</td>
<td style="text-align: right;">824</td>
<td style="text-align: right;">88.91</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Monterey</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">878.02</td>
<td style="text-align: right;">4963</td>
<td style="text-align: right;">958.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">849.64</td>
<td style="text-align: right;">5373</td>
<td style="text-align: right;">981.84</td>
</tr>
<tr class="even">
<td style="text-align: left;">Napa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">322.96</td>
<td style="text-align: right;">2312</td>
<td style="text-align: right;">402.93</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">304.90</td>
<td style="text-align: right;">1966</td>
<td style="text-align: right;">372.44</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Nevada</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">188.92</td>
<td style="text-align: right;">1004</td>
<td style="text-align: right;">211.56</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">178.45</td>
<td style="text-align: right;">1079</td>
<td style="text-align: right;">210.44</td>
</tr>
<tr class="even">
<td style="text-align: left;">Orange</td>
<td style="text-align: right;">18</td>
<td style="text-align: right;">6633.15</td>
<td style="text-align: right;">25898</td>
<td style="text-align: right;">5887.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">6444.60</td>
<td style="text-align: right;">23856</td>
<td style="text-align: right;">6117.61</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Placer</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">762.54</td>
<td style="text-align: right;">2903</td>
<td style="text-align: right;">681.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">740.55</td>
<td style="text-align: right;">3176</td>
<td style="text-align: right;">700.27</td>
</tr>
<tr class="even">
<td style="text-align: left;">Plumas</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">29.14</td>
<td style="text-align: right;">692</td>
<td style="text-align: right;">67.12</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">30.24</td>
<td style="text-align: right;">665</td>
<td style="text-align: right;">71.94</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Riverside</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">4256.84</td>
<td style="text-align: right;">16784</td>
<td style="text-align: right;">3648.29</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3993.04</td>
<td style="text-align: right;">15375</td>
<td style="text-align: right;">3662.45</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sacramento</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">2965.57</td>
<td style="text-align: right;">11618</td>
<td style="text-align: right;">2411.02</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2845.65</td>
<td style="text-align: right;">10954</td>
<td style="text-align: right;">2448.18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Benito</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">124.06</td>
<td style="text-align: right;">612</td>
<td style="text-align: right;">124.01</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">118.34</td>
<td style="text-align: right;">700</td>
<td style="text-align: right;">130.46</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Bernardino</td>
<td style="text-align: right;">13</td>
<td style="text-align: right;">3557.63</td>
<td style="text-align: right;">12792</td>
<td style="text-align: right;">2811.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3366.73</td>
<td style="text-align: right;">11907</td>
<td style="text-align: right;">2879.26</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Diego</td>
<td style="text-align: right;">154</td>
<td style="text-align: right;">7348.08</td>
<td style="text-align: right;">24432</td>
<td style="text-align: right;">6028.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">6990.02</td>
<td style="text-align: right;">26749</td>
<td style="text-align: right;">6273.51</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Francisco</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">2102.60</td>
<td style="text-align: right;">9775</td>
<td style="text-align: right;">2273.60</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2106.80</td>
<td style="text-align: right;">9971</td>
<td style="text-align: right;">2403.52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Joaquin</td>
<td style="text-align: right;">19</td>
<td style="text-align: right;">1390.77</td>
<td style="text-align: right;">4954</td>
<td style="text-align: right;">1089.58</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1214.32</td>
<td style="text-align: right;">5812</td>
<td style="text-align: right;">1067.10</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Luis Obispo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">532.08</td>
<td style="text-align: right;">2743</td>
<td style="text-align: right;">607.93</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">514.71</td>
<td style="text-align: right;">3246</td>
<td style="text-align: right;">623.91</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Mateo</td>
<td style="text-align: right;">25</td>
<td style="text-align: right;">1819.36</td>
<td style="text-align: right;">7592</td>
<td style="text-align: right;">1826.57</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1800.65</td>
<td style="text-align: right;">7280</td>
<td style="text-align: right;">1902.03</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Barbara</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">888.30</td>
<td style="text-align: right;">4951</td>
<td style="text-align: right;">1005.84</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">862.67</td>
<td style="text-align: right;">4865</td>
<td style="text-align: right;">1037.02</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Santa Clara</td>
<td style="text-align: right;">67</td>
<td style="text-align: right;">4613.35</td>
<td style="text-align: right;">34502</td>
<td style="text-align: right;">5363.89</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4656.38</td>
<td style="text-align: right;">31582</td>
<td style="text-align: right;">5435.47</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Cruz</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">598.43</td>
<td style="text-align: right;">3028</td>
<td style="text-align: right;">638.40</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">568.83</td>
<td style="text-align: right;">2923</td>
<td style="text-align: right;">626.68</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Shasta</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">241.30</td>
<td style="text-align: right;">1996</td>
<td style="text-align: right;">278.40</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">227.89</td>
<td style="text-align: right;">1986</td>
<td style="text-align: right;">271.82</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sierra</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5.01</td>
<td style="text-align: right;">133</td>
<td style="text-align: right;">14.03</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.95</td>
<td style="text-align: right;">131</td>
<td style="text-align: right;">13.95</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Siskiyou</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">64.40</td>
<td style="text-align: right;">1340</td>
<td style="text-align: right;">119.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">59.71</td>
<td style="text-align: right;">1204</td>
<td style="text-align: right;">111.23</td>
</tr>
<tr class="even">
<td style="text-align: left;">Solano</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">888.29</td>
<td style="text-align: right;">5038</td>
<td style="text-align: right;">867.09</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">801.01</td>
<td style="text-align: right;">4895</td>
<td style="text-align: right;">831.11</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sonoma</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">1115.95</td>
<td style="text-align: right;">4519</td>
<td style="text-align: right;">1163.69</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1086.50</td>
<td style="text-align: right;">5388</td>
<td style="text-align: right;">1193.46</td>
</tr>
<tr class="even">
<td style="text-align: left;">Stanislaus</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">994.70</td>
<td style="text-align: right;">4222</td>
<td style="text-align: right;">866.39</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">849.25</td>
<td style="text-align: right;">4503</td>
<td style="text-align: right;">810.82</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sutter</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">165.01</td>
<td style="text-align: right;">954</td>
<td style="text-align: right;">167.68</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">159.06</td>
<td style="text-align: right;">1393</td>
<td style="text-align: right;">197.10</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tehama</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">78.54</td>
<td style="text-align: right;">544</td>
<td style="text-align: right;">97.88</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">75.68</td>
<td style="text-align: right;">594</td>
<td style="text-align: right;">98.51</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Trinity</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16.96</td>
<td style="text-align: right;">327</td>
<td style="text-align: right;">37.85</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16.74</td>
<td style="text-align: right;">430</td>
<td style="text-align: right;">40.20</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tulare</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">722.46</td>
<td style="text-align: right;">3525</td>
<td style="text-align: right;">683.69</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">661.54</td>
<td style="text-align: right;">3855</td>
<td style="text-align: right;">695.52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tuolumne</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">89.62</td>
<td style="text-align: right;">1253</td>
<td style="text-align: right;">165.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">80.70</td>
<td style="text-align: right;">1203</td>
<td style="text-align: right;">161.74</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ventura</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">1751.57</td>
<td style="text-align: right;">8294</td>
<td style="text-align: right;">1660.59</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1702.70</td>
<td style="text-align: right;">6841</td>
<td style="text-align: right;">1698.75</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Yolo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">446.55</td>
<td style="text-align: right;">2276</td>
<td style="text-align: right;">457.24</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">426.10</td>
<td style="text-align: right;">2289</td>
<td style="text-align: right;">449.68</td>
</tr>
<tr class="even">
<td style="text-align: left;">Yuba</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">103.08</td>
<td style="text-align: right;">480</td>
<td style="text-align: right;">86.94</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">102.80</td>
<td style="text-align: right;">592</td>
<td style="text-align: right;">106.66</td>
</tr>
</tbody>
</table>

Based on the above table, the number progression from partially
vaccinated to fully vaccinated makes sense; there are overall less fully
vaccinated people than partially vaccinated people. In other words, to
be fully vaccinated means you reached partial vaccination at one point.
Additionally, partial vaccination has minimum values that extend beyond
0 while full vaccination does not. This reflects how someone cannot be
fully vaccinated immediately and needs a first dose to reach that status
eventually. No values appear abnormal in this table as either,
fortunately.

The third table includes data regarding vaccine dose counts and also is
stratified by vaccine company.

    mergedCA %>% 
      group_by(county) %>%
      summarise(Doses_min = min(total_doses), 
                Doses_mean = mean(total_doses),
                Doses_max = max(total_doses),
                Doses_sd = sd(total_doses),
                JJ_min = min(jj_doses),
                JJ_mean= mean(jj_doses),
                JJ_max = max(jj_doses),
                JJ_sd = sd(jj_doses),
                Mod_min = min(moderna_doses),
                Mod_mean= mean(moderna_doses),
                Mod_max = max(moderna_doses),
                Mod_sd = sd(moderna_doses),
                Pfi_min = min(pfizer_doses),
                Pfi_mean= mean(pfizer_doses),
                Pfi_max = max(pfizer_doses),
                Pfi_sd = sd(pfizer_doses)) %>% 
      knitr::kable(col.names = c("County", 
                                 "Min Doses", 
                                 "Mean Doses", 
                                 "Max Doses",
                                 "SD Doses",
                                 "Min JJ Doses", 
                                 "Mean JJ Doses", 
                                 "Max JJ Doses",
                                 "SD JJ Doses",
                                 "Min Mod Doses", 
                                 "Mean Mod Doses", 
                                 "Max Mod Doses",
                                 "SD Mod Doses",
                                 "Min Pfi Doses", 
                                 "Mean Pfi Doses", 
                                 "Max Pfi Doses",
                                 "SD Pfi Doses"), digits = 2, "pipe")

<table style="width:100%;">
<colgroup>
<col style="width: 7%" />
<col style="width: 4%" />
<col style="width: 5%" />
<col style="width: 4%" />
<col style="width: 4%" />
<col style="width: 5%" />
<col style="width: 6%" />
<col style="width: 5%" />
<col style="width: 5%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 5%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 6%" />
<col style="width: 5%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">County</th>
<th style="text-align: right;">Min Doses</th>
<th style="text-align: right;">Mean Doses</th>
<th style="text-align: right;">Max Doses</th>
<th style="text-align: right;">SD Doses</th>
<th style="text-align: right;">Min JJ Doses</th>
<th style="text-align: right;">Mean JJ Doses</th>
<th style="text-align: right;">Max JJ Doses</th>
<th style="text-align: right;">SD JJ Doses</th>
<th style="text-align: right;">Min Mod Doses</th>
<th style="text-align: right;">Mean Mod Doses</th>
<th style="text-align: right;">Max Mod Doses</th>
<th style="text-align: right;">SD Mod Doses</th>
<th style="text-align: right;">Min Pfi Doses</th>
<th style="text-align: right;">Mean Pfi Doses</th>
<th style="text-align: right;">Max Pfi Doses</th>
<th style="text-align: right;">SD Pfi Doses</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Alameda</td>
<td style="text-align: right;">41</td>
<td style="text-align: right;">7797.58</td>
<td style="text-align: right;">25711</td>
<td style="text-align: right;">6764.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">367.97</td>
<td style="text-align: right;">8037</td>
<td style="text-align: right;">1050.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2340.10</td>
<td style="text-align: right;">10209</td>
<td style="text-align: right;">2463.82</td>
<td style="text-align: right;">41</td>
<td style="text-align: right;">5089.51</td>
<td style="text-align: right;">15506.00</td>
<td style="text-align: right;">4085.26</td>
</tr>
<tr class="even">
<td style="text-align: left;">Alpine</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5.09</td>
<td style="text-align: right;">117</td>
<td style="text-align: right;">13.19</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.15</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.92</td>
<td style="text-align: right;">116</td>
<td style="text-align: right;">13.17</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.14</td>
<td style="text-align: right;">2.00</td>
<td style="text-align: right;">0.41</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Amador</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">129.16</td>
<td style="text-align: right;">986</td>
<td style="text-align: right;">157.75</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">254</td>
<td style="text-align: right;">16.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">85.47</td>
<td style="text-align: right;">906</td>
<td style="text-align: right;">141.60</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">38.69</td>
<td style="text-align: right;">233.00</td>
<td style="text-align: right;">33.27</td>
</tr>
<tr class="even">
<td style="text-align: left;">Butte</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">680.79</td>
<td style="text-align: right;">3374</td>
<td style="text-align: right;">691.56</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">27.23</td>
<td style="text-align: right;">482</td>
<td style="text-align: right;">58.55</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">304.42</td>
<td style="text-align: right;">2664</td>
<td style="text-align: right;">450.92</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">349.14</td>
<td style="text-align: right;">2729.00</td>
<td style="text-align: right;">352.83</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Calaveras</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">148.23</td>
<td style="text-align: right;">959</td>
<td style="text-align: right;">173.60</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.68</td>
<td style="text-align: right;">75</td>
<td style="text-align: right;">7.83</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">76.92</td>
<td style="text-align: right;">748</td>
<td style="text-align: right;">122.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">66.64</td>
<td style="text-align: right;">866.00</td>
<td style="text-align: right;">103.71</td>
</tr>
<tr class="even">
<td style="text-align: left;">Colusa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">69.87</td>
<td style="text-align: right;">445</td>
<td style="text-align: right;">83.38</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.65</td>
<td style="text-align: right;">71</td>
<td style="text-align: right;">7.93</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">45.79</td>
<td style="text-align: right;">393</td>
<td style="text-align: right;">71.17</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">21.43</td>
<td style="text-align: right;">141.00</td>
<td style="text-align: right;">18.62</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Contra Costa</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">5556.42</td>
<td style="text-align: right;">19247</td>
<td style="text-align: right;">4952.60</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">177.75</td>
<td style="text-align: right;">4629</td>
<td style="text-align: right;">453.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1545.27</td>
<td style="text-align: right;">7050</td>
<td style="text-align: right;">1769.50</td>
<td style="text-align: right;">20</td>
<td style="text-align: right;">3833.40</td>
<td style="text-align: right;">14804.00</td>
<td style="text-align: right;">3328.14</td>
</tr>
<tr class="even">
<td style="text-align: left;">Del Norte</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">76.49</td>
<td style="text-align: right;">500</td>
<td style="text-align: right;">87.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.17</td>
<td style="text-align: right;">77</td>
<td style="text-align: right;">9.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">41.76</td>
<td style="text-align: right;">351</td>
<td style="text-align: right;">62.57</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">30.55</td>
<td style="text-align: right;">175.00</td>
<td style="text-align: right;">34.73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">El Dorado</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">704.90</td>
<td style="text-align: right;">3264</td>
<td style="text-align: right;">614.43</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">27.12</td>
<td style="text-align: right;">458</td>
<td style="text-align: right;">48.82</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">320.72</td>
<td style="text-align: right;">1394</td>
<td style="text-align: right;">334.86</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">357.06</td>
<td style="text-align: right;">1875.00</td>
<td style="text-align: right;">328.95</td>
</tr>
<tr class="even">
<td style="text-align: left;">Fresno</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">3456.07</td>
<td style="text-align: right;">12603</td>
<td style="text-align: right;">2861.69</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">118.73</td>
<td style="text-align: right;">1426</td>
<td style="text-align: right;">231.58</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1381.40</td>
<td style="text-align: right;">7193</td>
<td style="text-align: right;">1589.35</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1955.95</td>
<td style="text-align: right;">6828.00</td>
<td style="text-align: right;">1294.03</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Glenn</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">87.56</td>
<td style="text-align: right;">672</td>
<td style="text-align: right;">109.55</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.54</td>
<td style="text-align: right;">255</td>
<td style="text-align: right;">16.86</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">42.20</td>
<td style="text-align: right;">457</td>
<td style="text-align: right;">76.12</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">40.83</td>
<td style="text-align: right;">349.00</td>
<td style="text-align: right;">48.17</td>
</tr>
<tr class="even">
<td style="text-align: left;">Humboldt</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">520.38</td>
<td style="text-align: right;">2311</td>
<td style="text-align: right;">504.53</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">25.76</td>
<td style="text-align: right;">470</td>
<td style="text-align: right;">49.52</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">218.57</td>
<td style="text-align: right;">2150</td>
<td style="text-align: right;">297.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">276.06</td>
<td style="text-align: right;">1581.00</td>
<td style="text-align: right;">282.52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Imperial</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">837.15</td>
<td style="text-align: right;">5050</td>
<td style="text-align: right;">814.96</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">51.40</td>
<td style="text-align: right;">1637</td>
<td style="text-align: right;">150.39</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">334.03</td>
<td style="text-align: right;">2375</td>
<td style="text-align: right;">391.16</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">451.72</td>
<td style="text-align: right;">3547.00</td>
<td style="text-align: right;">454.25</td>
</tr>
<tr class="even">
<td style="text-align: left;">Inyo</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">62.91</td>
<td style="text-align: right;">568</td>
<td style="text-align: right;">81.36</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.84</td>
<td style="text-align: right;">128</td>
<td style="text-align: right;">8.45</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">35.05</td>
<td style="text-align: right;">245</td>
<td style="text-align: right;">52.02</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">26.02</td>
<td style="text-align: right;">536.00</td>
<td style="text-align: right;">64.84</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Kern</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">2650.67</td>
<td style="text-align: right;">10386</td>
<td style="text-align: right;">2012.37</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">112.73</td>
<td style="text-align: right;">1382</td>
<td style="text-align: right;">231.50</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1083.16</td>
<td style="text-align: right;">4220</td>
<td style="text-align: right;">1022.68</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1454.79</td>
<td style="text-align: right;">5494.00</td>
<td style="text-align: right;">1015.12</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kings</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">373.58</td>
<td style="text-align: right;">1654</td>
<td style="text-align: right;">324.43</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">17.67</td>
<td style="text-align: right;">389</td>
<td style="text-align: right;">44.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">167.55</td>
<td style="text-align: right;">1558</td>
<td style="text-align: right;">212.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">188.37</td>
<td style="text-align: right;">1088.00</td>
<td style="text-align: right;">144.56</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lake</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">208.43</td>
<td style="text-align: right;">1122</td>
<td style="text-align: right;">214.92</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">9.50</td>
<td style="text-align: right;">476</td>
<td style="text-align: right;">33.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">128.11</td>
<td style="text-align: right;">1005</td>
<td style="text-align: right;">170.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">70.82</td>
<td style="text-align: right;">839.00</td>
<td style="text-align: right;">91.24</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lassen</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">46.83</td>
<td style="text-align: right;">1077</td>
<td style="text-align: right;">89.63</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.21</td>
<td style="text-align: right;">78</td>
<td style="text-align: right;">8.59</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">34.28</td>
<td style="text-align: right;">1065</td>
<td style="text-align: right;">87.89</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">8.34</td>
<td style="text-align: right;">86.00</td>
<td style="text-align: right;">12.71</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Los Angeles</td>
<td style="text-align: right;">410</td>
<td style="text-align: right;">42190.25</td>
<td style="text-align: right;">137449</td>
<td style="text-align: right;">33594.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1594.79</td>
<td style="text-align: right;">24711</td>
<td style="text-align: right;">3589.47</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16347.69</td>
<td style="text-align: right;">62780</td>
<td style="text-align: right;">16603.93</td>
<td style="text-align: right;">246</td>
<td style="text-align: right;">24247.77</td>
<td style="text-align: right;">71114.00</td>
<td style="text-align: right;">16040.87</td>
</tr>
<tr class="even">
<td style="text-align: left;">Madera</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">475.73</td>
<td style="text-align: right;">1969</td>
<td style="text-align: right;">408.95</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">18.37</td>
<td style="text-align: right;">358</td>
<td style="text-align: right;">39.28</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">196.26</td>
<td style="text-align: right;">1210</td>
<td style="text-align: right;">228.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">261.10</td>
<td style="text-align: right;">1338.00</td>
<td style="text-align: right;">226.24</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Marin</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1363.59</td>
<td style="text-align: right;">5901</td>
<td style="text-align: right;">1350.98</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">42.57</td>
<td style="text-align: right;">742</td>
<td style="text-align: right;">94.14</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">441.83</td>
<td style="text-align: right;">2582</td>
<td style="text-align: right;">576.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">879.20</td>
<td style="text-align: right;">4176.00</td>
<td style="text-align: right;">836.01</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mariposa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">47.41</td>
<td style="text-align: right;">499</td>
<td style="text-align: right;">79.17</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.00</td>
<td style="text-align: right;">39</td>
<td style="text-align: right;">4.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">33.90</td>
<td style="text-align: right;">488</td>
<td style="text-align: right;">75.89</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.52</td>
<td style="text-align: right;">121.00</td>
<td style="text-align: right;">14.11</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mendocino</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">359.00</td>
<td style="text-align: right;">2248</td>
<td style="text-align: right;">430.02</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.95</td>
<td style="text-align: right;">247</td>
<td style="text-align: right;">24.84</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">144.55</td>
<td style="text-align: right;">1958</td>
<td style="text-align: right;">262.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">202.49</td>
<td style="text-align: right;">1805.00</td>
<td style="text-align: right;">275.20</td>
</tr>
<tr class="even">
<td style="text-align: left;">Merced</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">817.73</td>
<td style="text-align: right;">3630</td>
<td style="text-align: right;">711.77</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">31.98</td>
<td style="text-align: right;">833</td>
<td style="text-align: right;">83.84</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">284.36</td>
<td style="text-align: right;">1460</td>
<td style="text-align: right;">296.15</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">501.39</td>
<td style="text-align: right;">2557.00</td>
<td style="text-align: right;">429.14</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Modoc</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">21.85</td>
<td style="text-align: right;">209</td>
<td style="text-align: right;">40.89</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.47</td>
<td style="text-align: right;">109</td>
<td style="text-align: right;">6.47</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">16.36</td>
<td style="text-align: right;">198</td>
<td style="text-align: right;">38.21</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.02</td>
<td style="text-align: right;">99.00</td>
<td style="text-align: right;">9.00</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mono</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">59.41</td>
<td style="text-align: right;">874</td>
<td style="text-align: right;">130.78</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.10</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">2.88</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">23.60</td>
<td style="text-align: right;">527</td>
<td style="text-align: right;">75.50</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">34.71</td>
<td style="text-align: right;">604.00</td>
<td style="text-align: right;">93.36</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Monterey</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1774.78</td>
<td style="text-align: right;">10353</td>
<td style="text-align: right;">1703.66</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">73.40</td>
<td style="text-align: right;">2237</td>
<td style="text-align: right;">188.12</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">807.61</td>
<td style="text-align: right;">4517</td>
<td style="text-align: right;">958.01</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">893.77</td>
<td style="text-align: right;">4942.00</td>
<td style="text-align: right;">837.65</td>
</tr>
<tr class="even">
<td style="text-align: left;">Napa</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">653.25</td>
<td style="text-align: right;">3105</td>
<td style="text-align: right;">659.75</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">18.87</td>
<td style="text-align: right;">380</td>
<td style="text-align: right;">43.88</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">277.07</td>
<td style="text-align: right;">1885</td>
<td style="text-align: right;">378.92</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">357.30</td>
<td style="text-align: right;">2244.00</td>
<td style="text-align: right;">357.75</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Nevada</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">383.60</td>
<td style="text-align: right;">1592</td>
<td style="text-align: right;">371.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">11.02</td>
<td style="text-align: right;">442</td>
<td style="text-align: right;">29.18</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">159.46</td>
<td style="text-align: right;">887</td>
<td style="text-align: right;">184.46</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">213.12</td>
<td style="text-align: right;">931.00</td>
<td style="text-align: right;">210.87</td>
</tr>
<tr class="even">
<td style="text-align: left;">Orange</td>
<td style="text-align: right;">18</td>
<td style="text-align: right;">13581.33</td>
<td style="text-align: right;">42936</td>
<td style="text-align: right;">10543.02</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">417.16</td>
<td style="text-align: right;">5607</td>
<td style="text-align: right;">844.82</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5122.90</td>
<td style="text-align: right;">19820</td>
<td style="text-align: right;">5081.05</td>
<td style="text-align: right;">17</td>
<td style="text-align: right;">8041.27</td>
<td style="text-align: right;">25101.00</td>
<td style="text-align: right;">5429.81</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Placer</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1574.87</td>
<td style="text-align: right;">4768</td>
<td style="text-align: right;">1208.37</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">43.23</td>
<td style="text-align: right;">781</td>
<td style="text-align: right;">80.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">587.71</td>
<td style="text-align: right;">2265</td>
<td style="text-align: right;">582.97</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">943.93</td>
<td style="text-align: right;">3033.00</td>
<td style="text-align: right;">672.01</td>
</tr>
<tr class="even">
<td style="text-align: left;">Plumas</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">60.38</td>
<td style="text-align: right;">716</td>
<td style="text-align: right;">104.69</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.93</td>
<td style="text-align: right;">286</td>
<td style="text-align: right;">19.65</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">46.26</td>
<td style="text-align: right;">711</td>
<td style="text-align: right;">100.09</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.19</td>
<td style="text-align: right;">141.00</td>
<td style="text-align: right;">14.94</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Riverside</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">8516.86</td>
<td style="text-align: right;">27774</td>
<td style="text-align: right;">6503.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">242.49</td>
<td style="text-align: right;">4771</td>
<td style="text-align: right;">497.53</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3128.41</td>
<td style="text-align: right;">12615</td>
<td style="text-align: right;">3190.01</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">5145.96</td>
<td style="text-align: right;">15628.00</td>
<td style="text-align: right;">3368.91</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sacramento</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">6038.00</td>
<td style="text-align: right;">18514</td>
<td style="text-align: right;">4415.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">172.18</td>
<td style="text-align: right;">3004</td>
<td style="text-align: right;">314.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2305.39</td>
<td style="text-align: right;">9727</td>
<td style="text-align: right;">2199.56</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">3560.43</td>
<td style="text-align: right;">10514.00</td>
<td style="text-align: right;">2327.58</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Benito</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">248.48</td>
<td style="text-align: right;">1038</td>
<td style="text-align: right;">225.95</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.81</td>
<td style="text-align: right;">258</td>
<td style="text-align: right;">28.13</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">93.17</td>
<td style="text-align: right;">597</td>
<td style="text-align: right;">109.10</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">144.50</td>
<td style="text-align: right;">748.00</td>
<td style="text-align: right;">146.55</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Bernardino</td>
<td style="text-align: right;">13</td>
<td style="text-align: right;">7101.39</td>
<td style="text-align: right;">22233</td>
<td style="text-align: right;">5218.34</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">215.44</td>
<td style="text-align: right;">3754</td>
<td style="text-align: right;">442.18</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2470.18</td>
<td style="text-align: right;">9191</td>
<td style="text-align: right;">2454.37</td>
<td style="text-align: right;">13</td>
<td style="text-align: right;">4415.77</td>
<td style="text-align: right;">11655.00</td>
<td style="text-align: right;">2753.18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Diego</td>
<td style="text-align: right;">175</td>
<td style="text-align: right;">14661.90</td>
<td style="text-align: right;">45105</td>
<td style="text-align: right;">10778.11</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">548.56</td>
<td style="text-align: right;">5917</td>
<td style="text-align: right;">899.63</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">5815.70</td>
<td style="text-align: right;">21632</td>
<td style="text-align: right;">5590.41</td>
<td style="text-align: right;">175</td>
<td style="text-align: right;">8297.64</td>
<td style="text-align: right;">26508.00</td>
<td style="text-align: right;">5561.53</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Francisco</td>
<td style="text-align: right;">9</td>
<td style="text-align: right;">4402.23</td>
<td style="text-align: right;">17069</td>
<td style="text-align: right;">4186.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">179.02</td>
<td style="text-align: right;">4491</td>
<td style="text-align: right;">464.64</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1328.51</td>
<td style="text-align: right;">6503</td>
<td style="text-align: right;">1465.42</td>
<td style="text-align: right;">9</td>
<td style="text-align: right;">2894.70</td>
<td style="text-align: right;">10132.00</td>
<td style="text-align: right;">2669.37</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Joaquin</td>
<td style="text-align: right;">19</td>
<td style="text-align: right;">2676.80</td>
<td style="text-align: right;">9241</td>
<td style="text-align: right;">1995.74</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">84.20</td>
<td style="text-align: right;">2173</td>
<td style="text-align: right;">242.29</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1004.95</td>
<td style="text-align: right;">3905</td>
<td style="text-align: right;">942.70</td>
<td style="text-align: right;">16</td>
<td style="text-align: right;">1587.65</td>
<td style="text-align: right;">5027.00</td>
<td style="text-align: right;">1071.26</td>
</tr>
<tr class="even">
<td style="text-align: left;">San Luis Obispo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1093.11</td>
<td style="text-align: right;">4676</td>
<td style="text-align: right;">1119.09</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">40.48</td>
<td style="text-align: right;">777</td>
<td style="text-align: right;">88.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">465.01</td>
<td style="text-align: right;">2574</td>
<td style="text-align: right;">538.52</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">587.62</td>
<td style="text-align: right;">2514.00</td>
<td style="text-align: right;">608.83</td>
</tr>
<tr class="odd">
<td style="text-align: left;">San Mateo</td>
<td style="text-align: right;">25</td>
<td style="text-align: right;">3784.02</td>
<td style="text-align: right;">13379</td>
<td style="text-align: right;">3336.20</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">134.31</td>
<td style="text-align: right;">2638</td>
<td style="text-align: right;">366.66</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1245.22</td>
<td style="text-align: right;">7482</td>
<td style="text-align: right;">1434.98</td>
<td style="text-align: right;">25</td>
<td style="text-align: right;">2404.49</td>
<td style="text-align: right;">8468.00</td>
<td style="text-align: right;">2001.24</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Barbara</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1810.86</td>
<td style="text-align: right;">8709</td>
<td style="text-align: right;">1753.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">76.02</td>
<td style="text-align: right;">1731</td>
<td style="text-align: right;">184.93</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">642.53</td>
<td style="text-align: right;">3160</td>
<td style="text-align: right;">683.30</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1092.31</td>
<td style="text-align: right;">5537.00</td>
<td style="text-align: right;">1130.13</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Santa Clara</td>
<td style="text-align: right;">67</td>
<td style="text-align: right;">9616.40</td>
<td style="text-align: right;">45081</td>
<td style="text-align: right;">9416.19</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">346.31</td>
<td style="text-align: right;">7737</td>
<td style="text-align: right;">935.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2937.50</td>
<td style="text-align: right;">15964</td>
<td style="text-align: right;">3236.22</td>
<td style="text-align: right;">20</td>
<td style="text-align: right;">6332.59</td>
<td style="text-align: right;">34818.00</td>
<td style="text-align: right;">6445.78</td>
</tr>
<tr class="even">
<td style="text-align: left;">Santa Cruz</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">1202.80</td>
<td style="text-align: right;">4826</td>
<td style="text-align: right;">1125.48</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">31.06</td>
<td style="text-align: right;">482</td>
<td style="text-align: right;">62.53</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">559.76</td>
<td style="text-align: right;">2674</td>
<td style="text-align: right;">629.43</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">611.98</td>
<td style="text-align: right;">3049.00</td>
<td style="text-align: right;">601.47</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Shasta</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">480.95</td>
<td style="text-align: right;">3064</td>
<td style="text-align: right;">459.43</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">24.93</td>
<td style="text-align: right;">861</td>
<td style="text-align: right;">61.15</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">226.98</td>
<td style="text-align: right;">2329</td>
<td style="text-align: right;">278.09</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">229.03</td>
<td style="text-align: right;">2489.00</td>
<td style="text-align: right;">283.38</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sierra</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">10.11</td>
<td style="text-align: right;">193</td>
<td style="text-align: right;">25.11</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">0.19</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">0.68</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">8.59</td>
<td style="text-align: right;">192</td>
<td style="text-align: right;">25.16</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1.33</td>
<td style="text-align: right;">12.83</td>
<td style="text-align: right;">2.27</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Siskiyou</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">128.49</td>
<td style="text-align: right;">1700</td>
<td style="text-align: right;">190.73</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">3.73</td>
<td style="text-align: right;">64</td>
<td style="text-align: right;">8.82</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">68.21</td>
<td style="text-align: right;">674</td>
<td style="text-align: right;">97.49</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">56.56</td>
<td style="text-align: right;">1198.00</td>
<td style="text-align: right;">138.75</td>
</tr>
<tr class="even">
<td style="text-align: left;">Solano</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">1749.13</td>
<td style="text-align: right;">6688</td>
<td style="text-align: right;">1475.05</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">47.73</td>
<td style="text-align: right;">874</td>
<td style="text-align: right;">96.97</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">698.68</td>
<td style="text-align: right;">4878</td>
<td style="text-align: right;">941.25</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1002.72</td>
<td style="text-align: right;">5117.00</td>
<td style="text-align: right;">737.87</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sonoma</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">2294.88</td>
<td style="text-align: right;">7990</td>
<td style="text-align: right;">2112.92</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">71.88</td>
<td style="text-align: right;">1378</td>
<td style="text-align: right;">150.90</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">898.14</td>
<td style="text-align: right;">4062</td>
<td style="text-align: right;">1105.39</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1324.86</td>
<td style="text-align: right;">4768.00</td>
<td style="text-align: right;">1081.48</td>
</tr>
<tr class="even">
<td style="text-align: left;">Stanislaus</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1895.88</td>
<td style="text-align: right;">6554</td>
<td style="text-align: right;">1512.97</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">61.80</td>
<td style="text-align: right;">1981</td>
<td style="text-align: right;">184.65</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">694.88</td>
<td style="text-align: right;">3821</td>
<td style="text-align: right;">737.46</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1139.20</td>
<td style="text-align: right;">4251.00</td>
<td style="text-align: right;">879.22</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sutter</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">331.45</td>
<td style="text-align: right;">1700</td>
<td style="text-align: right;">300.83</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">14.69</td>
<td style="text-align: right;">633</td>
<td style="text-align: right;">54.88</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">168.77</td>
<td style="text-align: right;">1161</td>
<td style="text-align: right;">205.22</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">147.99</td>
<td style="text-align: right;">885.00</td>
<td style="text-align: right;">135.24</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tehama</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">156.42</td>
<td style="text-align: right;">779</td>
<td style="text-align: right;">155.94</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">6.11</td>
<td style="text-align: right;">73</td>
<td style="text-align: right;">10.23</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">89.27</td>
<td style="text-align: right;">708</td>
<td style="text-align: right;">115.01</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">61.04</td>
<td style="text-align: right;">492.00</td>
<td style="text-align: right;">83.57</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Trinity</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">34.11</td>
<td style="text-align: right;">463</td>
<td style="text-align: right;">61.48</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">2.31</td>
<td style="text-align: right;">110</td>
<td style="text-align: right;">8.61</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">23.76</td>
<td style="text-align: right;">389</td>
<td style="text-align: right;">58.00</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">8.04</td>
<td style="text-align: right;">38.00</td>
<td style="text-align: right;">6.68</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tulare</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1420.37</td>
<td style="text-align: right;">5437</td>
<td style="text-align: right;">1147.32</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">38.81</td>
<td style="text-align: right;">605</td>
<td style="text-align: right;">81.27</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">507.26</td>
<td style="text-align: right;">2333</td>
<td style="text-align: right;">541.17</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">874.30</td>
<td style="text-align: right;">3450.00</td>
<td style="text-align: right;">666.22</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tuolumne</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">175.55</td>
<td style="text-align: right;">1465</td>
<td style="text-align: right;">255.26</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">4.23</td>
<td style="text-align: right;">47</td>
<td style="text-align: right;">6.70</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">117.72</td>
<td style="text-align: right;">1406</td>
<td style="text-align: right;">223.42</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">53.59</td>
<td style="text-align: right;">639.00</td>
<td style="text-align: right;">97.85</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ventura</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">3568.08</td>
<td style="text-align: right;">11898</td>
<td style="text-align: right;">3059.02</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">123.56</td>
<td style="text-align: right;">2170</td>
<td style="text-align: right;">274.63</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">1449.72</td>
<td style="text-align: right;">6923</td>
<td style="text-align: right;">1637.01</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">1994.81</td>
<td style="text-align: right;">5970.00</td>
<td style="text-align: right;">1428.03</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Yolo</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">905.61</td>
<td style="text-align: right;">3876</td>
<td style="text-align: right;">814.17</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">26.42</td>
<td style="text-align: right;">565</td>
<td style="text-align: right;">63.55</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">411.30</td>
<td style="text-align: right;">2229</td>
<td style="text-align: right;">468.59</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">467.89</td>
<td style="text-align: right;">1995.00</td>
<td style="text-align: right;">386.56</td>
</tr>
<tr class="even">
<td style="text-align: left;">Yuba</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">209.50</td>
<td style="text-align: right;">795</td>
<td style="text-align: right;">161.47</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">12.75</td>
<td style="text-align: right;">491</td>
<td style="text-align: right;">42.08</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">109.89</td>
<td style="text-align: right;">614</td>
<td style="text-align: right;">113.31</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">86.85</td>
<td style="text-align: right;">410.00</td>
<td style="text-align: right;">62.59</td>
</tr>
</tbody>
</table>

This table provides the most unique information to me compared to the
other two. Again, minimums and maximums for these values do not appear
to deviate from normal, expected values. Because the two data sets used
in this analysis were downloaded from reputable, government agencies,
the data quality is reliable. Already there are clear differences in
vaccine company means, with Pfizer most often administered, Moderna
second, and Johnson & Johnson third. Visualizing this through pie charts
will be interesting to see how the total doses are shared by company for
each county.

#### Figures

<u>Figure 1: Vaccination Rates Over Time by County</u>

The following plots shows how administered vaccine doses have
accumulated since December 2020. Each county has its own plot, and the
y-variable was standardized earlier to make counties’ data comparable.

    mergedCA %>%
      ggplot(data, mapping = aes(x = date, y = dose_standard, 
                                 color = county)) +
      geom_line() + 
      xlab("Doses per Person") +
      facet_wrap(vars(county), ncol = 5) +
      theme(legend.position = "none", strip.background = element_blank(),
            strip.text = element_text(size = rel(0.8), margin = margin()),
            panel.spacing = unit(3, "pt")) +
      labs(x = "Date", y = "Cumulative Total Doses (Standardized by Population Size)")

![](AbercrombieJaxon_Midterm_PM566_files/figure-markdown_strict/fig1-1.png)

<u>Figure 2: Percent Fully Vaccinated by County</u>

The following figure shows the percent of individuals fully vaccinated
by county in descending order.

    mergedCA <-
      mergedCA %>%
      group_by(county) %>%
      mutate(maxPerc = max(perc_vaccinated)/310)

    mergedCA %>%
      ggplot(aes(y = reorder(county, maxPerc), x = maxPerc)) +
      geom_bar(stat = "identity") +
      labs(y = "County", x = "Percent Fully Vaccinated by 10/20/2021") +
      theme(legend.position = "none")

![](AbercrombieJaxon_Midterm_PM566_files/figure-markdown_strict/fig2-1.png)

<u>Figure 3: Pie Chart of Vaccines Administered Based on Company across
Counties</u>

The following pie charts demonstrate how the administered doses vary by
vaccine company for each county.

    companyVax <- 
      mergedCA %>%
      group_by(county) %>%
      summarise(maxJJ = 
                  (max(cumulative_jj_doses)/max(cumulative_total_doses))*100,
                maxMod = 
                  (max(cumulative_moderna_doses)/max(cumulative_total_doses))*100,
                maxPfi = 
                  (max(cumulative_pfizer_doses)/max(cumulative_total_doses)*100))

    county <- companyVax$county
    JJ <- companyVax$maxJJ
    Moderna <- companyVax$maxMod
    Pfizer <- companyVax$maxPfi
    df <- data.frame(county, JJ, Moderna, Pfizer)

    require(tidyr)
    companyVax <- gather(df, variable,value, -county)

    companyVax %>%
      ggplot(aes(x="", y=value, fill=variable)) +
      geom_bar(stat="identity") +
      coord_polar("y", start=0) +
      facet_wrap(vars(county), ncol = 8) +
      labs(x = "", y = "", legend = "Vaccine Company") +
      theme(axis.text.x=element_blank()) +
      scale_fill_brewer("Blues")

![](AbercrombieJaxon_Midterm_PM566_files/figure-markdown_strict/fig4-1.png)

<u>Figure 4: Comparing Trends in Vaccination to Trends in COVID-19 Cases
and Deaths (All of CA)</u>

The following three figures are aligned vertically so that trends can be
acknowledged based on date. The first plot depicts cases since the start
of vaccine roll-out. The second plot depicts deaths since the start of
vaccine roll-out. The third plot depicts the total number of doses
administered per day since the start of vaccine roll-out. Focus will be
placed on the month of July and onward.

    pCases <-
      mergedCA %>%
      ggplot(mapping = aes(x = date, 
                           y = cases)) +
      geom_line() + 
      xlab("") +
      theme(legend.position = "none", strip.background = element_blank(),
            strip.text = element_text(size = rel(0.8), margin = margin()),
            panel.spacing = unit(3, "pt")) +
      labs(x = "Date", y = "Cases")

    pDeaths <-
      mergedCA %>%
      ggplot(mapping = aes(x = date, 
                           y = deaths)) +
      geom_line() + 
      xlab("") +
      theme(legend.position = "none", strip.background = element_blank(),
            strip.text = element_text(size = rel(0.8), margin = margin()),
            panel.spacing = unit(3, "pt")) +
      labs(x = "Date", y = "Deaths")
      
      
    pDoses <-  
      mergedCA %>%
      ggplot(mapping = aes(x = date, 
                           y = total_doses)) +
      geom_line() + 
      xlab("") +
      theme(legend.position = "none", strip.background = element_blank(),
            strip.text = element_text(size = rel(0.8), margin = margin()),
            panel.spacing = unit(3, "pt")) +
      labs(x = "Date", y = "Total Doses Administered")
      

    plot_grid(pDeaths, pCases, pDoses, 
              ncol = 1, 
              labels = c("Cases", "Deaths", "Total Doses"))

![](AbercrombieJaxon_Midterm_PM566_files/figure-markdown_strict/fig5-1.png)

## Conclusion

Conclusions to the primary question and two secondary questions of this
data project were found.

In regard to vaccination differences across California counties, it is
evident that:

-   All counties follow an S-like curve when looking at cumulative
    vaccine doses, though steepness differs (Figure 1). The S-shape
    explains a surge in vaccination rates, with cumulative dose counts
    plateauing at extremes. This is explainable by eligibility, since at
    the start of vaccine roll-out (Jan-March 2021), not many people
    could get vaccinated—just the elderly and healthcare officials. Now,
    past July 2021, we are seeing a plateau because most individuals who
    wanted vaccine doses received them.

    -   After around April, when eligibility to get a vaccine continued
        to widen, a greater amount of individuals were able to receive
        their vaccine and did so. Notably steep increases in vaccination
        appear to take place in counties like Napa, Santa Clara, Alpine,
        Alameda, and San Francisco, while significantly less steep
        increases appear in counties like Yuba, Kern, and Modoc.

        -   This generally seems to portray that more urban or suburban
            counties have greater rates of fully vaccinated individuals
            than those that have rural settings.

    -   Regardless of trends by county, there have been general
        increases in doses being administered each day despite the surge
        of getting vaccinated slowing down.

-   The majority of counties in Northern California (Marin, San
    Francisco, Santa Clara) have a greater proportion of fully
    vaccination individuals than countries in Central (Tulare, Fresno)
    and Southern California (San Bernardino, Riverside) based on
    Figure 2.

    -   The lowest percentage of fully vaccinated individuals belongs to
        Lassen County, with about 25% of individuals fully vaccinated.
        On the other hand, Marin County has the most with about 78%.

    -   Roughly 25% of counties have more than 60% of their populations
        fully vaccinated, and around 50% of counties between 45% and 60%
        are fully vaccinated. Overall the numbers are low compared to
        the state’s goals.

-   Much of these differences could be most attributable to vaccine
    access and/or political affiliation. Since more rural areas likely
    face travel-related obstacles to receive vaccines and more
    conservative areas likely hold anti-vaccine sentiment, it seems
    reasonable that these trends exist in the data.

    -   Of the two, vaccine access may pose a larger threat. This is
        because many of the counties in the lower half of Figure 2 are
        the farthest from urban centers and likely have less vaccination
        services within their counties.

When assessing whether specific vaccine companies were more prevalent in
some areas than others, the pie charts produced tell an intriguing story
about whether Moderna or Pfizer dominate certain counties.

-   The third summary statistic table demonstrates that Pfizer doses are
    given most often, Moderna second, and Johnson & Johnson third.

-   Based on Figure 3, we can see that Moderna and Pfizer essentially
    take turns with being a county’s most popular vaccine. Johnson &
    Johnson, even when acknowledging that it is a single-dose vaccine,
    consistently makes up small portions of the charts. Despite Johnson
    & Johnson being easier with one dose, individuals still appear to
    receive other companies’ vaccines more often.

-   It appears that the Pfizer vaccine dominates in the most well-known
    urban centers like Los Angeles, San Francisco, and San Diego
    Counties while Moderna is the more common vaccine in rural places
    like Sierra, Shasta, Plumas, and Lake Counties—just to name a few
    (Figure 3).

-   This difference in vaccine prevalence between urban and rural areas
    is neat to see. It is possible that this could be attributed to
    Pfizer being the first vaccine released and the most populous/urban
    counties having robust vaccination centers to serve their people
    with Pfizer. Pfizer being the first available vaccine would
    significantly impact the number of individuals who have it since it
    had great demand. Also, since Moderna has longer wait periods
    between doses and Johnson & Johnson was removed from the market at
    one point, Pfizer’s domination is even more reasonable.

When assessing whether trends in cases and deaths potentially caused
trends in vaccine, there is a slight increase in vaccinations when
variants were widespread over the summer of 2021 based on Figure 4.

-   It appears that the increase in deaths and cases between July and
    August could have influenced the visible increase in vaccine doses
    administered during the same time and up until now, October 2021.
    Because the summer was a great demonstration of how vaccines protect
    people and prevent severe illness and death, it makes sense that
    people who originally were skeptical changed their mind.

-   It appears that another small vaccination surge happened in October,
    and any increase in vaccination is beneficial. Especially as we gear
    up for winter months, increasing the percent of individuals
    vaccinated is important.

-   Now that booster shots are becoming a hot topic, it is unclear
    whether the dose surge in October could be attributed to booster
    shots or people receiving their first or second doses.

Overall, this was an extremely exciting project to pursue. Especially as
the motivation of unvaccinated individuals to get vaccinated is more
important each day, identifying which counties can improve on
vaccination is crucial. The tables and plots produced can also help
answer several other COVID-related questions, and they assisted in
finding trends related to vaccine companies and state-wide vaccination
trends that are likely influenced by cases and deaths. In the future, it
would be neat to incorporate data about booster shots, do this analysis
on a country-wide scale, analyze how COVID-19 fluctuates this winter,
and produce maps of California with the information and even more unique
visualizations.
