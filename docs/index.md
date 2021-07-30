---
title: "Tidycensus in R"
author: "Fang Fang"
knit: (function(input_file, encoding) {
  out_dir <- 'docs';
  rmarkdown::render(input_file,
 encoding=encoding,
 output_file=file.path(dirname(input_file), out_dir, 'index.html'))})
output: 
  html_document: 
    keep_md: yes
---





In this lab exercise, we will explore how to download the datasets using `tidycensus` package from the API.


## Basic usage of `tidycensus`

The `tidycensus` package [(See details here)](https://walkerke.github.io/tidycensus/) downloads data directly from the U.S. Census Bureau's application programming interface (API). You can still use [data.census.gov](https://data.census.gov) website interface to manually select and download data step by step.The benefit of using `tidycensus` package is that the data pipeline is self-contained and reproducible. 

Here is the API handbook from the official [**website**](https://www.census.gov/content/dam/Census/data/developers/api-user-guide/api-guide.pdf)

Before you start `tidycensus` downloading process, you need the following items: 

* A Census API key (if you don't already have one, please request and active one from the website [**here**](https://api.census.gov/data/key_signup.html)). Save your API key properly. You will need it later. 
* Places/regions of interest to look up (e.g., all 52 states, or over thousands of counties)
* The census year you are interested. 
* The codes/number of the tables of interests (population? education? income? etc.)

Two main functions in `tidycensus`:

- `get_decennial()` is designed to obtain data and feature geometry for the decennial Census which is conducted **every ten** years (e.g., 1990, 2000, 2010, and currently working on census 2020) in the US.
- `get_acs()` is designed to obtain data and feature geometry for the **five-year** American Community Survey, which is conducted continuously, and which focuses on describing the characteristics of the population for the years in between the decennial census. 



# Example 1
For more information, please visit [**Basic usage of tidycensus**.](https://walkerke.github.io/tidycensus/articles/basic-usage.html)

1. Load the package and type your census api key. 

```r
# Your Code Here
library(tidycensus)
census_api_key("Put your key here")
```
2. Let's request a dataset from the US Census website. The geography level we are interested in is "state", the variable is population (code: B01003_001) from ACS in 2017. Name the dataset `pop_2017.` Below is the detailed description of each argument in the function. 

* *geography = "state"* specifies the geogrpahy which we want - in this case estimates for each state.
* *variables = c("B01003_001")*  specifies which variable we want to download. The code for total population is "B01003_001". You need to specify the codes as character string or vector of character strings of variable IDs. It automatically returns the estimate (column ends in E) and the margin of error (column ends in M) associated with the variable.
* *year = 2017*  The year, or endyear, of the ACS sample. 2009 through 2018 are available. Here we set year as 2017. 
* *survey="acs5"* The ACS contains one-year, three-year, and five-year surveys expressed as "acs1", "acs3", and "acs5". The default selection is "acs5." So we are requesting data from 2013-2017.
* *output="wide"* each row represents an enumeration unit and the variables are in the columns.
* *geometry=T*: default is false. We set as true to keep the spatial component of the dataset in order to create maps. 

Please execute the command below to request this population data from census API. 


```r
pop_2017<-get_acs(geography = "state", variables =c("B01003_001"), 
                  year=2017, survey="acs5", output="wide", geometry=T)
```



The census data includes a number of different geographic types. Click [**here**](https://www2.census.gov/geo/pdfs/reference/geodiagram.pdf) for more information.

<img src="image/WB_3_Geogs.png" width="500px" />

3. Let's examine this dataset pop_2017.It has 52 rows (states) and 4 columns. 

- **GEOID:** numeric codes that uniquely identify all administrative/legal and statistical geographic areas. Here we are using the GEOID at state level. 
- **NAME:** name of each state
- **Column ends with E:**  estimates (in this case population estimates), with each column representing a *row* in the data table we pulled up on [data.census.gov](https://data.census.gov/cedsci/table?q=&g=0100000US.04000.001&table=B01001&tid=ACSDT5Y2017.B01001&d=ACS%205-Year%20Estimates%20Detailed%20Tables&vintage=2017&y=2017&t=Populations%20and%20People&hidePreview=false&cid=B01001_001E&lastDisplayedRow=27).
- **Column ends with M:** the margin of error associated with each estimate.

<img src="image/pop_2017.png" width="921" />

4. To check out the information of all the variables, execute the scripts below. We set `cache = TRUE` to save the variable descriptions as a dataframe on your computer for future access. You can click on `table_code` dataset now. 


```r
table_code <- load_variables(2017, "acs5", cache = TRUE)
```



# Example 2
Let's request another dataset from the Census website. The geography level we are interested in this time is "tract", variable is still population (code:B01003_001) from ACS in 2017. But we are only interested in data within IL. You can specify state="IL" in the function. Name the dataset `pop_2017_IL_tract`. Please execute the command below to request this population data at tract level from census API. 



```r
pop_2017_IL_tract <- get_acs(geography = "tract", state = "IL", variables = c("B01003_001"), 
                             year=2017, survey="acs5", output="wide")
```

This pop_2017_IL_tract dataset includes 4 columns.  "B01003_001E" is the population estimation, and "B01003_001M" is the margin of error. 

# Example 3
Let's download race/ethnicity by county in Illinois for 2010 using `get_decennial` function. 

We need to check out the variables in decennial census data first. You can also check out the [**website**](https://projects.ncsu.edu/woodlands/current_pdfs/teleconf_data_sources.pdf) for more information. 

Note: it might take 3-5 minutes to load variables this time. 



```r
table_code_decennial <- load_variables(2010, "sf1", cache = TRUE)
```

Assume we need these 4 variables: 

Code  | Description
------------- | -------------
P005003 | White alone
P005004  | Black or African American alone
P005006| Asian alone
P004003| Hispanic or Latino

Since we need to extract multiple variables this time, we can first save these variables in a single dataset called `vars`. We can also assign column names instead of variable codes. Then we can directly call this `vars` within the `get_decennial` function. Please execute the command below to request this population data at county level from census API. 



```r
vars <- c(White="P005003", Black_or_African_American="P005004", 
          Asian="P005006", Hispanic_or_Latino="P004003")

IL_decennial <- get_decennial(geography = "county", variables = vars, 
                              year = 2010, state = "IL", output="wide")
```


This `IL_decennial` dataset includes 102 rows (counties) and 6 variables. All the column names are pre-defined so it is much more straightforward! 


That wasn't *so* hard, right? If you are interested in getting data from USCENSUS in the future:

1. Do you need ACS data, or data from decennial census?

2. Make sure you know what tables or variables you are looking for. Check out the `table_code`first. 

2. Make sure what is the geography level of interest.

Once you execute the scripts, R will download the dataset and it is immediately available as a data frame in your R Environment. 








