# Google Data Analytic Capstone - Case Study 1
## Introduction
This repository contains my approaches and works to solve problems of the Google Data Analytic Case Study. The full document and related details can be found on [Google Data Analytics Capstone: Complete a Case Study](https://www.coursera.org/learn/google-data-analytics-capstone) Course.

In order to complete the case study, the steps of the data analysis process (ask, prepare, process, analyze, share, and act) will be followed and each part contains its own roadmap (Guiding questions, Key tasks and deliverable).

<br>

## Ask
### - What is the problem you are trying to solve?
To find out the bike usage differences between casual riders and annual members and the digital media influent on them.
### - How can your insights drive business decisions?
The insights can help Cyclistic to determine strategies to help converting casual riders to annual members (increase more annual members).
### * A clear statement of the business task
Find out differences between casual riders and annual members, and how could digital media influence them differently.

<br>

## Prepare
### - Where is your data located?
Google Data Analytic Course provided the [link](https://divvy-tripdata.s3.amazonaws.com/index.html) for the Cyclistic bike sharing infomation dataset.
### - How is the data organized?
The data is separated in month with its own zip file.
### - Are there issues with bias or credibility in this data? Does your data ROCCC?
There are no issues with bias or credibility since the population are all the clients(causal riders and annual members) of Cyclistic. The data is ROCCC because it's because it's reliable, original, comprehensive, current and cited.
### - How are you addressing licensing, privacy, security, and accessibility?
The dataset is under this [license](https://ride.divvybikes.com/data-license-agreement). It's private and secure since the dataset doesn't contain any personal infomation of the clients.
### - How did you verify the data’s integrity?
The dataset contain consistent columns in each zip file and data type are accurate in each column.
### - How does it help you answer your question?
Insights about riders and their riding style, riding time can be found to answer the questions.
### - Are there any problems with the data?
Limited general personal information(age, sex, etc.). Besides, large amounts of missing stations related info (start/end station name, station id)
### * A description of all data sources used
The data that will be use to analyze and identify trends are the Cyclistic trip data from the previous 12 months (2020-12 to 2021-11).

<br>

## Process
*tidyverse* packages will be used in this project. 
```
install.packages("tidyverse") 
library(tidyverse)
```
<br>

### Merging multiple csv files into one data frame
```
merged_data <- list.files(path = "/Users/katechen/Desktop/case_study_dataset",     # Identify all csv files in folder
                       pattern = "*.csv", full.names = TRUE) %>% 
  lapply(read_csv) %>%                                                             # Store all files in list
  bind_rows                                                                        # Combine data sets into one data set 
head(merged_data)    
data_all <- as.data.frame(merged_data)                                             # Convert tibble into a data frame
```

<br>

### Data Cleaning
#### - Removing duplicates
Check and remove duplicate riders id record
```
sum(table(data_all$ride_id)-1)                                                     # Check numbers of duplicated record
unique_data <- data_all[!duplicated(data_all$ride_id), ]                           # Remove duplicated riders id record                                                   
print(paste("Removed", nrow(data_all) - nrow(unique_data), "duplicated rows"))     # Print out duplicated rows to make sure numbers of duplicates
```
<img width="255" alt="Screen Shot 2021-12-26 at 10 36 40 PM" src="https://user-images.githubusercontent.com/94023835/147431858-f21fc091-469d-4702-bf1f-c9e3108294bc.png">

Since the numbers of duplicated record is 0 and printed message is "Removed 0 duplicated rows", it means no duplicated records in the data set.


<br>


#### - String inconsistencies
Checking typos, capitalization errors, misplaced punctuation, or similar character data errors.
```
unique(unique_data$rideable_type)
unique(unique_data$member_casual)
```
<img width="380" alt="Screen Shot 2021-12-26 at 10 42 06 PM" src="https://user-images.githubusercontent.com/94023835/147432149-ad835ab7-caaa-42fd-af7b-81e22acaebd3.png">

There are no typos or data errors in this two columns, all values are consistence.


<br>


#### - Missing Value
Checking numbers if missing values and percentage of missing values per variable
```
sum(is.na(unique_data))
apply(unique_data, 2, function(col)sum(is.na(col))/length(col))
```
<img width="1247" alt="Screen Shot 2021-12-26 at 10 49 06 PM" src="https://user-images.githubusercontent.com/94023835/147432478-95fffa95-17f7-418c-b63e-79976996f078.png">

Large amount of missing value exist in station related observations, deleting them directly can lead to a inaccurate and less representative result. Therefore, instead of removing them, it is better to analyze them as misisng value group.


<br>


#### - Parsing datetime columns
```
unique_data$started_at <- as.POSIXct(unique_data$started_at, "%Y-%m-%d %H:%M:%S")
unique_data$ended_at <- as.POSIXct(unique_data$ended_at, "%Y-%m-%d %H:%M:%S")
```

<br>


#### - Splitting Date and Time into new columns
```
unique_data <- unique_data %>%
  mutate(started_date = format(unique_data$started_at, format="%Y-%m-%d"))

unique_data <- unique_data %>%
  mutate(started_time = format(unique_data$started_at, format="%H:%M:%S"))

unique_data <- unique_data %>%
  mutate(ended_date = format(unique_data$ended_at, format="%Y-%m-%d"))

unique_data <- unique_data %>%
  mutate(ended_time = format(unique_data$ended_at, format="%H:%M:%S"))
```


<br>


#### - Making sure Ended Date is later(bigger) or at the same day(equal) to the Started Date
```
sum(unique_data$started_date > unique_data$ended_date)
cleaned_data <- subset(unique_data, (unique_data$started_date > unique_data$ended_date) == FALSE)
```
As it shown above in, 378 of rows contain a Started Date that is later than Ended Date, which doesn't make sense. Those observations would be dropped from the data set. And the data set now have 5478718 remaining rows.


<br>


#### - Mutate a new row *ride_length_mins*
It represents the riding length of each riders in minutes.
```
cleaned_data <- cleaned_data %>%
  mutate(ride_length_mins = as.numeric(difftime(cleaned_data$ended_at, cleaned_data$started_at, units="mins")))
summary(cleaned_data$ride_length_mins)
```
<img width="403" alt="Screen Shot 2021-12-26 at 11 12 25 PM" src="https://user-images.githubusercontent.com/94023835/147433609-770afbd2-ead0-4209-9b09-369978f36461.png">


<br>


#### - Mutate a new row *Weekday*
This represents the day of the week that each ride started.
```
cleaned_data <- cleaned_data %>%
  mutate(weekday = paste(strftime(cleaned_data$started_at, "%a")))
unique(cleaned_data$weekday)
```
<img width="343" alt="Screen Shot 2021-12-26 at 11 13 31 PM" src="https://user-images.githubusercontent.com/94023835/147433666-c07b2b52-0bee-48a4-a039-6ee20bf4280a.png">


<br>


#### - Mutate a new row *started_hour*
Extracting the hour of the day that each ride started.
```
cleaned_data <- cleaned_data %>%
  mutate(start_hour = strftime(cleaned_data$ended_at, "%H"))
unique(cleaned_data$start_hour)
```
<img width="909" alt="Screen Shot 2021-12-26 at 11 16 34 PM" src="https://user-images.githubusercontent.com/94023835/147433846-c1dc074b-14f2-4716-90d8-2b7ee625d059.png">

<br>

### Saving data as csv. file
```
cleaned_data %>%
  write.csv("cyclistic_clean_data.csv")
```

<br>


### - What tools are you choosing and why?
Since this is a large data set, using R is more effective.
### - Have you ensured your data’s integrity?
Yes, the data set is consistent and accurate.
### - What steps have you taken to ensure that your data is clean?
Checking duplicated and missing values, make sure values are consistence and removed rows that contains incorrect srated time.
### How can you verify that your data is clean and ready to analyze?
1. Going back to the original unclean data set and comparing it to the one I have now.
2. Think about the bussine problem again and make sure the varaibles in the data set are useful to help solving the problem and project goal.
3. Have teammate to review the data from a fresh persepective (if possible).
### Have you documented your cleaning process so you can review and share those results?
Yes, it is all documented in R markdown and this  repository.


<br>


## Analyze
