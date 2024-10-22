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
#### All data contain a group of data from 1 to 40. Each group contains data from the Excel file for each sheet. The previously run code was used to create a  new column in each sheet that contains the sheet name. With this trick, each row contains the month name in the last column. 
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
````
View(combined_data)
`````
![Combined_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/06.png)
## New Table created
The two important column needed for the analysis is the prescribed drugs and the MonthYear column
````
combined_dispensed_drug <- combined_data[, c("prescribed_drugs", "MonthYear")]
````
![Combined_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/07.png)
### CHeck unique values 
````
unique(combined_dispensed_drugs)
````
**2161** unique values in the combined dispensed drug 

## Convert all text to a drug name.
This is an important step in the data cleaning process to correct misspellings, remove extra space, and give the same drugs the same name to ensure consistency as "ACT", "Arthemeter", and "Antimalaria" drugs should all be represented with one unique name. All drugs cleaned to ensure consistency
````
combined_dispensed_drug$prescribed_drugs <- ifelse(grepl("PARA|PCM", combined_dispensed_drug$prescribed_drugs), 
+"PCM", 
+combined_dispensed_drug$prescribed_drugs)
````
#### Removed any word that contains TAB, CAP, 5,3,2
````
combined_dispensed_drug$prescribed_drugs <- gsub("\\b\\w*(TAB|CAP|5|2)\\w*\\b", "", combined_dispensed_drug$prescribed_drugs)
````
````
combined_dispensed_drug$prescribed_drugs <- gsub("\\b\\w*(TAB|CAP|STAT|START|3|8|1|4|6|7|9|5|2)\\w*\\b", "", combined_dispensed_drug$prescribed_drugs)
````
#### Remove Space
````
combined_dispensed_drug$prescribed_drugs <- trimws(combined_dispensed_drug$prescribed_drugs)
````
**1900** unique values remain at this stage

##### Replace entire cell content with "PCM" if it contains "PARA" or "PCM"
````
combined_dispensed_drug$prescribed_drugs <- ifelse(grepl("PARA|PCM", combined_dispensed_drug$prescribed_drugs),  "PCM", combined_dispensed_drug$prescribed_drugs) 
````
**1805** unique value at this stage

#### The above formula was then applied to other drugs to ensure consistency

- Ampicilline - Ampici
- Albendazole - Abend, Albe
- Strepsil - Strep
- Septrim - Sept
- Vitamin BCO - B.co, Complex, BCO, B. CO
- Ibuprofen - Ibu, Profen
- Gestid - Gest
- Metronidazole - Flagy, Metro
- Cough Syrup - Cough, 
- T.T - T.T, Tetanus, TT
- Diclofenac - Diclo
- Piriton - Piri, Priton
- Vitamin C - Vit.C, VITC, MINC MIN. C, Vitamin, Victam, Vit C
- Amlodipine - Amlo
- Nifedipine - Nif, Nefid, Nedi
- CO-Codamol - Cocod, Codam
- Omeprazole - Omep, Omipr
- Ampiclox - Ampicl, Ampcl, 
- Loperamide - Lop, Lapera, 
- ACT - ACT, ARTH
- Moduretic - Bond, Modu
- Eye drop - Eye
- Ear drop - Ear 
- Infusion - Normal, Saline, I.V.F, IVF, 
- Piriton - Chlorapheniramine
- Ciprofloxacin - Cipro
- Loratadine - Lora
- Amoxyl - Amox
- Folic Acid - Folic, Acid

The unique value reduces to **1190**

- ALBR for Albendazole
- VIT B for Vitamin BCO 
- PCKM for PCM 
- IBRU for Ibuprofen 
- Mist for Gestid
- Flayl for Metronidazole 
- AMIVC, CEE, VIT C, VIT. C, VIY.C, TAMIV.C, for vitamin C
- ANLO for Amlodipine 
- Omeo for Omeprazole
- Plox for Ampiclox 
- A CT for ACT
- BUDRE for Moduretic
- Amilod for Amlodipine
- Ampici for Ampicilline
- Loara for Loratadine, 
- Mist for GESTID 
- Bufen for Ibuprofen
- AMPXL for  Amoxicilline
- Metrinidazole for Metronidazole 

**1133** unique values are now observed. 
#### Remove row with "NA"
````
combined_dispensed_drug <- combined_dispensed_drug[complete.cases(combined_dispensed_drug), ]
````
**1096** unique values after removing "NA"
## Calculations
### Create a new table 
This new table contains the percentage of each drugs and the number of times each drugs are dispensed. 
````
percentage_table <- combined_dispensed_drug %>%
+    group_by(prescribed_drugs) %>%
+    summarise(count = n()) %>%
+    mutate(percentage = (count / sum(count)) * 100) %>%
+    arrange(desc(percentage))
`````
##### view(percentage_table)
![Drug_ercentage_table](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/12.png)

##### Two columns selected
````
selected_percentage_table <- percentage_table %>%
+    select(prescribed_drugs, percentage)
````
![Drug_ercentage_table](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/08.png)

##### Select subset and add together as others
````
percentage_subset <- selected_percentage_table$percentage[20:nrow(selected_percentage_table)]
> Others <- sum(percentage_subset, na.rm = TRUE)
````
##### Create a new row then add "Others" 
````
new_row <- data.frame(prescribed_drugs = "Others", percentage = Others)
````
````
final1_dispensed_drug <- rbind(selected_percentage_table, new_row)
````
````
print(final1_dispensed_drug)
````
Remove 20 to 170 in data as Others is added which contains the sum of 20 to 170 rows then crosscheck which is confirmed to be 100%
````
backup_final1_dispensed_drug <- final1_dispensed_drug
````
````
final1_dispensed_drug <- final1_dispensed_drug[-(20:170), ]
````
````
view(final1_dispensed_drug)
````
````
view(sum(final1_dispensed_drug$percentage, na.rm = TRUE))
````

## Create Chart




