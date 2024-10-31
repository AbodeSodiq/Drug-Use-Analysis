## Overview
This analysis was conducted over 33 months to understand drug usage in the clinic. The patients are adults, with no children or pediatrics. The goal is to identify patterns in drug consumption and patient demographics, which can help optimize drug inventory and improve patient care strategies.

## Objective
- Understand the usage pattern of drugs
- Identify the top 20 most prescribed medications.
- Analyze the prescription rate of each drug.
- Maintain sustainability of drugs in the clinic without running out before the next request.
- To improve employees' health and well-being.

## Importing of Data into R
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
#### Import all data
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
#### All data contain a list of data from 1 to 40. Each list contains data from the Excel file for each sheet. The previously run code was used to create a  new column in each sheet that contains the sheet name. Each row contains the month name in the last column with this code. 
````
view(all_data(1))
````
![All Data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/02.png)

## Data Cleaning 
### Extra Column and Unwanted List
The imported list contains a different number of columns which are 2, 6, 7,8, and 9 columns. After observation, The minimum number of columns should be 5, and maximum number of columns should be 7. Extra list(sheet) and unwanted columns are removed. 

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

### Convert all text to CAPITAL LETTER
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
### Clean and standardized column name 
#### Remove empty spaces
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
### Combine All Lists of Data Together
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
After combining all lists of data into one, it was observed that some new columns was created due to inconsistencies in column names such as prescribed drugs, v, patient name, prescribe drug, name
````
combined_data <- combined_data %>% mutate( prescribed_drugs = coalesce(prescribed_drugs, prescribed_drug) ) %>% select(-prescribed_drug)
````
````
View(combined_data)
`````
![Combined_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/06.png)
### New Table created
The two important columns needed for the analysis are the prescribed drugs and the MonthYear column
````
combined_dispensed_drug <- combined_data[, c("prescribed_drugs", "MonthYear")]
````
![Combined_data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/07.png)
### Check unique values 
Unique values in the dispensed drugs list were observed to identify errors or inconsistencies. 
````
unique(combined_dispensed_drugs)
````
**2161** Unique values in the combined dispensed drug were identified. It was observed that some drugs a represented with different names, presence of extra space, unnecessary figures etc

### Convert all text to a drug name.
This is an important step in the data cleaning process to correct misspellings, remove extra space, and give the same drugs the same name to ensure consistency. For example, "ACT", "Arthemeter", and "Antimalaria" drugs should all be represented with one unique name. 
````
combined_dispensed_drug$prescribed_drugs <- ifelse(grepl("PARA|PCM", combined_dispensed_drug$prescribed_drugs), 
+"PCM", 
+combined_dispensed_drug$prescribed_drugs)
````
Drugs cleaned to ensure consistency

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

## Calculations and Charts
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
##### Create a scattered plot 
````
ggplot(percentage_table, aes(x = `prescribed_drugs`, y = `count`)) +
    geom_point() +
    labs(title = "Scatter Plot of Prescribed Drugs vs. Count", y = "Count") +
    theme_minimal() +
    theme(axis.title.x = element_blank(),       
          axis.text.x = element_blank(),      
          axis.ticks.x = element_blank())    
`````
![Count_Scatered_plot](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Charts/18.png)

##### Two columns selected
````
selected_percentage_table <- percentage_table %>%
+    select(prescribed_drugs, percentage)
````
![Drug_percentage_table](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/08.png)

##### Select a subset and add together as others
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
### Create a bar chart 
````
ggplot(final1_dispensed_drug, aes(x = `prescribed_drugs`, y = `percentage`)) +
    geom_bar(stat = "identity") +
    labs(title = "Bar Chart of Prescribed Drugs vs. Percentage", 
         x = "Prescribed Drug", 
         y = "Percentage (%)") +
    theme_minimal() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
````
![Percentage_bar_chart](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Charts/19.png)

## Conclusions
- PCM (Paracetamol) and Ibuprofen, accounting for a combined 41.11%, dominate the prescription landscape, indicating a strong preference for pain management drugs.
- Antimalarials (ACT and others), despite being crucial in regions affected by malaria, only make up 9.31% of total prescriptions.
- A significant disparity exists between the top 10 most prescribed drugs and the remaining 160, suggesting heavy reliance on a small subset of medications.
- The "Others" category represents 150 drugs that collectively make up a very small fraction (0.06% per drug) of the total, showing a lack of diversity in prescriptions.
- Amoxicillin, important antibiotics, appear less frequently than expected, indicating either appropriate use of antibiotics or underdiagnosis of bacterial infections.
- Drugs such as Folic Acid and Vitamin C are prescribed at low rates, suggesting minimal focus on preventive treatments and supplementation.
Cough syrups and antacids (e.g., Omeprazole) have lower prescription rates, possibly reflecting a low incidence of respiratory or gastrointestinal conditions.
- The high prescription rate for Metronidazole suggests a notable incidence of gastrointestinal infections and conditions requiring this antibiotic.
- Anti-allergy drugs like Loratadine have low prescription rates, indicating either fewer allergy cases or potential underreporting.
- "Other drugs" represent the vast majority of the drug list but are prescribed at a much lower frequency, raising questions about stock management or underuse.

## Recommendations
- Monitor the high usage of pain relievers, particularly PCM and Ibuprofen, to ensure they are not overprescribed, potentially leading to dependency or side effects.
- Evaluate the need for more balanced drug prescriptions by exploring the underuse of the remaining 150+ drugs, which may help diversify treatment options for patients.
- Focus on increasing awareness of antimalarials in regions where malaria is prevalent, ensuring these drugs are readily available and prescribed appropriately.
- Encourage preventive care by promoting the prescription of multivitamins like Vitamin C and Folic Acid, especially in populations at risk of malnutrition or deficiency.
- Optimize stock and availability of the top 20 drugs to prevent shortages, while considering reducing excessive reliance on them.
- Promote antibiotic stewardship, particularly for drugs like Amoxicillin, to prevent the development of drug resistance due to overuse.
- Reassess the low use of antacids and cough syrups to ensure that conditions requiring these medications are appropriately diagnosed and treated.
- Investigate the low prescription rate of "Other drugs" to ensure they are still needed, possibly adjusting the inventory based on real patient needs.
- Promote allergy awareness and treatment options, including increasing the use of antihistamines like Loratadine in cases where allergies are underdiagnosed.
- Review the clinical guidelines and prescribing habits to identify areas where lesser-prescribed medications can play a role in improving patient outcomes.

## For inquiry,
- Email: abodesodiq195@gmail.com
- Tel: 08145313364
- LinkedIn: [Click here](https://www.linkedin.com/in/abode-sodiq-19b80418a/)

_I help healthcare brands & healthcare professionals to achieve their goals. Either to gain visibility, build trust, or increase income, I have helped over 150+ to achieve this. Your brand is next.
**Contact me today to get started**_

# THANK YOU






