### Walmart Store Sales Analysis
Walmart Inc. is an American multinational retail corporation that operates a chain of hypermarkets (also called supercenters), discount department stores, and grocery stores from the United States, headquartered in Bentonville, Arkansas
### Data Description
You are provided with historical sales data for 45 Walmart stores located in different regions. Each store contains a number of departments, and you are tasked with predicting the department-wide sales for each store.
In addition, Walmart runs several promotional markdown events throughout the year. These markdowns precede prominent holidays, the four largest of which are the Super Bowl, Labor Day, Thanksgiving, and Christmas. The weeks including these holidays are weighted five times higher in the evaluation than non-holiday weeks. Part of the challenge presented by this competition is modeling the effects of markdowns on these holiday weeks in the absence of complete/ideal historical data.
stores.csv: This file contains anonymized information about the 45 stores, indicating the type and size of store.
train.csv: This is the historical training data, which covers to 2010-02-05 to 2012-11-01. Within this file you will find the following fields:
Store - the store number Dept - the department number Date - the week Weekly_Sales - sales for the given department in the given store IsHoliday - whether the week is a special holiday week. 
test.csv: This file is identical to train.csv, except we have withheld the weekly sales. You must predict the sales for each triplet of store, department, and date in this file.
features.csv: This file contains additional data related to the store, department, and regional activity for the given dates. It contains the following fields:
Store - the store number Date - the week Temperature - average temperature in the region Fuel_Price - cost of fuel in the region MarkDown1-5 - anonymized data related to promotional markdowns that Walmart is running. MarkDown data is only available after Nov 2011, and is not available for all stores all the time. Any missing value is marked with an NA. 
CPI - the consumer price index Unemployment - the unemployment rate IsHoliday - whether the week is a special holiday week For convenience, the four holidays fall within the following weeks in the dataset (not all holidays are in the data):
Super Bowl: 12-Feb-10, 11-Feb-11, 10-Feb-12, 8-Feb-13 Labor Day: 10-Sep-10, 9-Sep-11, 7-Sep-12, 6-Sep-13 Thanksgiving: 26-Nov-10, 25-Nov-11, 23-Nov-12, 29-Nov-13 Christmas: 31-Dec-10, 30-Dec-11, 28-Dec-12, 27-Dec-13
Here I start my project by following steps:

```markdown
### Importing all the Required data and libraries
import pandas as pd
import numpy as np
from google.colab import drive
drive.mount('gdrive')
import os

data_path = '/content/gdrive/My Drive/mid_term_project/Walmart_store_sales_dataset'
train = pd.read_csv(os.path.join(data_path, 'train.csv'))
test = pd.read_csv(os.path.join(data_path, 'test.csv'))
stores = pd.read_csv(os.path.join(data_path, 'stores.csv'))
sampleSubmission = pd.read_csv(os.path.join(data_path, 'sampleSubmission.csv'))
features = pd.read_csv(os.path.join(data_path, 'features.csv'))

```
### Data Preprocessing
#### Handling Missing Values
At first we checked each data whether it has missing values or not.
```markdown
print('Train')
print(train.isnull().sum())
print('Test')
print(test.isnull().sum())
print('Stores')
print(stores.isnull().sum())
print('SampleSubmission')
print(sampleSubmission.isnull().sum())
print('Features')
print(features.isnull().sum())                      
```
Here we have null values in Features dataset's columns i.e MarkDown1, MarkDown2, MarkDown3, MarkDown4, MarkDown5, CPI and Unemployment. So now we have to check percentage of the missing values for each column.
```markdown
missing_percentage = features.isnull().sum() * 100 / len(features)
missing_value_df = pd.DataFrame({'percent_missing': missing_percentage})
missing_value_df
```
Here MarkDown1, MarkDown2, MarkDown3, MarkDown4, MarkDown5 columns have more than 50% null values so we remove it and use those columns which have less than 50% null values i.e CPI and Unemployment. Here we fill or impute missing values by using Simple Imputer() method *which* takes following arguments i.e missing_values, strategy and fill_value. Here we use mean as a strategy.

```markdown
features.drop(['MarkDown1','MarkDown2','MarkDown3', 'MarkDown4', 'MarkDown5'], axis = 1, inplace =True)
from sklearn.impute import SimpleImputer
cpi_impute = SimpleImputer(missing_values = np.nan, strategy = 'mean')
cpi_impute = imputer.fit(features[['CPI']])
features['CPI'] = cpi_impute.transform(features[['CPI']]).ravel()

from sklearn.impute import SimpleImputer
unemployment_impute = SimpleImputer(missing_values= np.nan, strategy = 'mean')
unemployment_impute = unemployment_impute.fit(features[['Unemployment']])
features['Unemployment'] = unemployment_impute.transform(feat_stores[['Unemployment']]).ravel()
```
Merging data                                                                     
Here we have merge two dataframes i.e stores and features on the basis of Store column

```markdown
feat_store = pd.merge(features,stores, on = 'Store', how = 'inner')
feat_store
```
In Data Analysis the date field must have datatime type so here we can see Date field has string type so we should convert them to  datetime type of all dataframes.
```markdown
feat_store.dtypes
pd.DataFrame({'Train_Type':train.dtypes, 'Test_Type':test.dtypes})
feat_store['Date'] = pd.to_datetime(feat_store['Date'])
train['Date'] = pd.to_datetime(train['Date'])
test['Date'] = pd.to_datetime(test['Date'])
```










