## Data Visualization 
#### Raw data visualized to understand the nature of the data in Excel
![Raw Data Exported](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/17.png)
## Import data in R
````
library(readxl)
library(dplyr)
````
#### File path to Excel file created
````
file_path <- "clinic_dispensed_drug.xlsx"
````
#### To import all data
````
sheets <- excel_sheets(file_path)

all_data <- lapply(sheets, function(sheet) {
  data <- read_excel(file_path, sheet = sheet)
  data$MonthYear <- sheet
  return(data)
})
````
#### View imported data
````
print(all_data)
````
![All Data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/01.png)
#### All data contain a group of data from 1 to 40. Each group contains data from the Excel file for each sheet. The previously run code was used to create a  new column in each sheet which contains the sheet name. With this trick, each row contains the month name in the last column. 
````
view(all_data(1))
````
![All Data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/02.png)
## Extra Column and Unwanted List
#### The imported list contains 2, 6, 7,8, and 9 columns. After observation, The minimum number of columns should be 5 and maximum number of columns should be 7. Extra list(sheet) and unwanted columns are removed. 
#### Indices of the sheets to remove specified
````
unwanted_indices <- c(6, 19, 32)
````
Unwanted sheets were removed using the indices
````
filtered_drugs_data <- drugs_all_data[-unwanted_indices]
````
#### Names of the remaining sheets checked
````
view(filtered_drugs_data)
````

![Filtered_drug_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/04.png)
## Convert all text to CAPITAL LETTER
````
convert_text_to_upper <- function(df) {
  df <- df %>% mutate(across(where(is.character), toupper))
  return(df)
}
````
###### Apply function
````
filtered_drugs_data <- lapply(filtered_drugs_data, convert_text_to_upper)
````
## Clean and standardized column name 
### Remove empty spaces
````
clean_column_names <- function(df) {
    colnames(df) <- gsub(" ", "_", colnames(df))
    return(df)
 }
````
##### Then
````
filtered_drugs_data <- lapply(filtered_drugs_data, clean_column_names)
````
## Combine All Lists of Data Together
##### function
````
convert_to_character <- function(df) {
  df[] <- lapply(df, as.character)
  return(df)
}
````
##### then apply
````
filtered_drugs_data <- lapply(filtered_drugs_data, convert_to_character)
````
### Combine all data frames into one
````
combined_data <- bind_rows(filtered_drugs_data_char)
````
### Merge column
A new column was created due to inconsistencies in column names such as prescribed drugs, v, patient name, prescribe drug, name
````
combined_data <- combined_data %>% mutate( prescribed_drugs = coalesce(prescribed_drugs, prescribed_drug) ) %>% select(-prescribed_drug)
````
View(combined_data)
![Combined_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/06.png)
Important column New combined dispensed drug created
combined_dispensed_drug <- combined_data[, c("prescribed_drugs", "MonthYear")]

2161 uniq values in the combined dispensed drug 

