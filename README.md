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
### All data imported
````
sheets <- excel_sheets(file_path)

all_data <- lapply(sheets, function(sheet) {
  data <- read_excel(file_path, sheet = sheet)
  data$MonthYear <- sheet
  return(data)
})
````
## All data imported to R
````
print(all_data)
````
![All Data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/01.png)
#### All data contain a group of data from 1 to 40. Each group contains data for each sheet from the Excel file. The previously run code was used to create a  new column in each sheet which contains the sheet name. With this trick, each row contains the month name in the last column. 
````
view(all_data(1))
````
![All Data](https://github.com/AbodeSodiq/Drug-Use-Analysis/blob/main/Tables/02.png)
