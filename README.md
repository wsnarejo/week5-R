# week5-R
Analysis of Global COVID-19 Pandemic Data
Estimated time needed: 90 minutes

Overview:
There are 10 tasks in this final project. All tasks will be graded by your peers who are also completing this assignment within the same session.

You need to submit the following the screenshot for the code and output for each task for review.

If you need to refresh your memories about specific coding details, you may refer to previous hands-on labs for code examples.

# If you are working on your local Jupyter notebook, please uncomment the below code and install the packages

install.packages("httr")
install.packages("xml2")
# If you are working on your local Jupyter notebook, please uncomment the below code and install the packages
​
install.packages("httr")
install.packages("xml2")
Updating HTML index of packages in '.Library'
Making 'packages.html' ... done
Updating HTML index of packages in '.Library'
Making 'packages.html' ... done
require("httr")
require("rvest")
library(httr)
library(rvest)
​
​
Loading required package: httr
Loading required package: rvest
Loading required package: xml2
Note: if you can import above libraries, please use install.packages() to install them first.

TASK 1: Get a COVID-19 pandemic Wiki page using HTTP request
First, let's write a function to use HTTP request to get a public COVID-19 Wiki page.

Before you write the function, you can open this public page from this

URL https://en.wikipedia.org/w/index.php?title=Template:COVID-19_testing_by_country using a web browser.

The goal of task 1 is to get the html page using HTTP request (httr library)

​
get_wiki_covid19_page <- function() {
    
  # Our target COVID-19 wiki page URL is: https://en.wikipedia.org/w/index.php?title=Template:COVID-19_testing_by_country  
  # Which has two parts: 
    # 1) base URL `https://en.wikipedia.org/w/index.php  
    # 2) URL parameter: `title=Template:COVID-19_testing_by_country`, seperated by question mark ?
​
  # Wiki page base
  wiki_base_url <- "https://en.wikipedia.org/w/index.php"
  # You will need to create a List which has an element called `title` to specify which page you want to get from Wiki
  # in our case, it will be `Template:COVID-19_testing_by_country`
​
  # - Use the `GET` function in httr library with a `url` argument and a `query` arugment to get a HTTP response
​
  # Use the `return` function to return the response
​
wiki_params <- list(title = "Template:COVID-19_testing_by_country")
​
wiki_response <- GET(wiki_base_url, query = wiki_params)
​
return(wiki_response)
    
}
​
​
Call the get_wiki_covid19_page function to get a http response with the target html page

# Call the get_wiki_covid19_page function and print the res
​
wiki_covid19_page_response <- get_wiki_covid19_page()
print(wiki_covid19_page_response)
Response [https://en.wikipedia.org/w/index.php?title=Template%3ACOVID-19_testing_by_country]
  Date: 2023-05-04 04:30
  Status: 200
  Content-Type: text/html; charset=UTF-8
  Size: 433 kB
<!DOCTYPE html>
<html class="client-nojs vector-feature-language-in-header-enabled vector-fea...
<head>
<meta charset="UTF-8"/>
<title>Template:COVID-19 testing by country - Wikipedia</title>
<script>document.documentElement.className="client-js vector-feature-language...
"75a68b1c-7276-4019-bf29-b09fe6a4a5be","wgCSPNonce":false,"wgCanonicalNamespa...
"CS1 uses Khmer-language script (km)","CS1 Khmer-language sources (km)","CS1 ...
"CS1 Mongolian-language sources (mn)","CS1 foreign language sources (ISO 639-...
"levels":1}}},"wgVisualEditor":{"pageLanguageCode":"en","pageLanguageDir":"lt...
...
TASK 2: Extract COVID-19 testing data table from the wiki HTML page
On the COVID-19 testing wiki page, you should see a data table <table> node contains COVID-19 testing data by country on the page:

Image
Note the numbers you actually see on your page may be different from above because it is still an on-going pandemic when creating this notebook.

The goal of task 2 is to extract above data table and convert it into a data frame

Now use the read_html function in rvest library to get the root html node from response

# Get the root html node from the http response in task 1 
wiki_covid19_page_root_node <- read_html(wiki_covid19_page_response)
Get the tables in the HTML root node using html_nodes function.

# Get the table node from the root html node
wiki_covid19_page_table_node <- html_node(wiki_covid19_page_root_node,"table")
Read the specific table from the multiple tables in the table_node using the html_table function and convert it into dataframe using as.data.frame

Hint:- Please read the table_node with index 2(ex:- table_node[2]).

# Read the table node and convert it into a data frame, and print the data frame for review
wiki_covid19_page_data_frame <- html_table(wiki_covid19_page_table_node)
wiki_covid19_page_data_frame 
​
covid_df <- as.data.frame(html_table(wiki_covid19_page_table_node[2]))
​
covid_df
​
A data.frame: 1 × 2
X1	X2
<lgl>	<chr>
NA	This template needs to be updated. Please help update this template to reflect recent events or newly available information. Relevant discussion may be found on the talk page.
Error in UseMethod("html_table"): no applicable method for 'html_table' applied to an object of class "list"
Traceback:

1. as.data.frame(html_table(wiki_covid19_page_table_node[2]))
2. html_table(wiki_covid19_page_table_node[2])
TASK 3: Pre-process and export the extracted data frame
The goal of task 3 is to pre-process the extracted data frame from the previous step, and export it as a csv file

Let's get a summary of the data frame

# Print the summary of the data frame
summary(wiki_covid19_page_data_frame)
    X1               X2           
 Mode:logical   Length:1          
 NA's:1         Class :character  
                Mode  :character  
As you can see from the summary, the columns names are little bit different to understand and some column data types are not correct. For example, the Tested column shows as character.

As such, the data frame read from HTML table will need some pre-processing such as removing irrelvant columns, renaming columns, and convert columns into proper data types.

We have prepared a pre-processing function for you to conver the data frame but you can also try to write one by yourself

preprocess_covid_data_frame <- function(data_frame) {
    
    shape <- dim(data_frame)
​
    # Remove the World row
    data_frame<-data_frame[!(data_frame$`Country.or.region`=="World"),]
    # Remove the last row
    data_frame <- data_frame[1:172, ]
    
    # We dont need the Units and Ref columns, so can be removed
    data_frame["Ref."] <- NULL
    data_frame["Units.b."] <- NULL
    
    # Renaming the columns
    names(data_frame) <- c("country", "date", "tested", "confirmed", "confirmed.tested.ratio", "tested.population.ratio", "confirmed.population.ratio")
    
    # Convert column data types
    data_frame$country <- as.factor(data_frame$country)
    data_frame$date <- as.factor(data_frame$date)
    data_frame$tested <- as.numeric(gsub(",","",data_frame$tested))
    data_frame$confirmed <- as.numeric(gsub(",","",data_frame$confirmed))
    data_frame$'confirmed.tested.ratio' <- as.numeric(gsub(",","",data_frame$`confirmed.tested.ratio`))
    data_frame$'tested.population.ratio' <- as.numeric(gsub(",","",data_frame$`tested.population.ratio`))
    data_frame$'confirmed.population.ratio' <- as.numeric(gsub(",","",data_frame$`confirmed.population.ratio`))
    
    return(data_frame)
}
​
Call the preprocess_covid_data_frame function

# call `preprocess_covid_data_frame` function and assign it to a new data frame
new_covid_data_frame <- preprocess_covid_data_frame(wiki_covid19_page_data_frame)
head(new_covid_data_frame)
Error in names(data_frame) <- c("country", "date", "tested", "confirmed", : 'names' attribute [7] must be the same length as the vector [2]
Traceback:

1. preprocess_covid_data_frame(wiki_covid19_page_data_frame)
Get the summary of the processed data frame again

# Print the summary of the processed data frame again
summary(new_covid_data_frame)
Error in summary(new_covid_data_frame): object 'new_covid_data_frame' not found
Traceback:

1. summary(new_covid_data_frame)
After pre-processing, you can see the columns and columns names are simplified, and columns types are converted into correct types.

The data frame has following columns:

country - The name of the country
date - Reported date
tested - Total tested cases by the reported date
confirmed - Total confirmed cases by the reported date
confirmed.tested.ratio - The ratio of confirmed cases to the tested cases
tested.population.ratio - The ratio of tested cases to the population of the country
confirmed.population.ratio - The ratio of confirmed cases to the population of the country
OK, we can call write.csv() function to save the csv file into a file.

# Export the data frame to a csv file
Note for IBM Waston Studio, there is no traditional "hard disk" associated with a R workspace.

Even if you call write.csv() method to save the data frame as a csv file, it won't be shown in IBM Cloud Object Storage asset UI automatically.

However, you may still check if the covid.csv exists using following code snippet:

# Get working directory
wd <- getwd()
# Get exported 
file_path <- paste(wd, sep="", "/covid.csv")
# File path
print(file_path)
file.exists(file_path)
Optional Step: If you have difficulties finishing above webscraping tasks, you may still continue with next tasks by downloading a provided csv file from here:

## Download a sample csv file
# covid_csv_file <- download.file("https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-RP0101EN-Coursera/v2/dataset/covid.csv", destfile="covid.csv")
# covid_data_frame_csv <- read.csv("covid.csv", header=TRUE, sep=",")
TASK 4: Get a subset of the extracted data frame
The goal of task 4 is to get the 5th to 10th rows from the data frame with only country and confirmed columns selected

# Read covid_data_frame_csv from the csv file
covid_data_frame_csv <- read.csv("covid.csv", header=TRUE, sep=",")
​
# Get the 5th to 10th rows, with two "country" "confirmed" columns
​
Warning message in file(file, "rt"):
“cannot open file 'covid.csv': No such file or directory”
Error in file(file, "rt"): cannot open the connection
Traceback:

1. read.csv("covid.csv", header = TRUE, sep = ",")
2. read.table(file = file, header = header, sep = sep, quote = quote, 
 .     dec = dec, fill = fill, comment.char = comment.char, ...)
3. file(file, "rt")
TASK 5: Calculate worldwide COVID testing positive ratio
The goal of task 5 is to get the total confirmed and tested cases worldwide, and try to figure the overall positive ratio using confirmed cases / tested cases

# Get the total confirmed cases worldwide
​
# Get the total tested cases worldwide
​
# Get the positive ratio (confirmed / tested)
​
TASK 6: Get a country list which reported their testing data
The goal of task 6 is to get a catalog or sorted list of countries who have reported their COVID-19 testing data

# Get the `country` column
​
# Check its class (should be Factor)
​
# Conver the country column into character so that you can easily sort them
​
# Sort the countries AtoZ
​
# Sort the countries ZtoA
​
# Print the sorted ZtoA list
​
TASK 7: Identify countries names with a specific pattern
The goal of task 7 is using a regular expression to find any countires start with United

# Use a regular expression `United.+` to find matches
​
# Print the matched country names
​
TASK 8: Pick two countries you are interested, and then review their testing data
The goal of task 8 is to compare the COVID-19 test data between two countires, you will need to select two rows from the dataframe, and select country, confirmed, confirmed-population-ratio columns

# Select a subset (should be only one row) of data frame based on a selected country name and columns
​
​
# Select a subset (should be only one row) of data frame based on a selected country name and columns
​
TASK 9: Compare which one of the selected countries has a larger ratio of confirmed cases to population
The goal of task 9 is to find out which country you have selected before has larger ratio of confirmed cases to population, which may indicate that country has higher COVID-19 infection risk

# Use if-else statement
# if (check which confirmed.population value is greater) {
#    print()
# } else {
#    print()
# }
​
TASK 10: Find countries with confirmed to population ratio rate less than a threshold
The goal of task 10 is to find out which countries have the confirmed to population ratio less than 1%, it may indicate the risk of those countries are relatively low

# Get a subset of any countries with `confirmed.population.ratio` less than the threshold
​

