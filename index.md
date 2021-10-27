### Walmart Store Sales Analysis
Walmart Inc. is an American multinational retail corporation that operates a chain of hypermarkets (also called supercenters), discount department stores, and grocery stores from the United States, headquartered in Bentonville, Arkansas
### Data Description
The data collected ranges from 2010 to 2012, where 45 Walmart stores across the country were included in this analysis. Each store contains several departments, and we are tasked with predicting the department-wide sales for each store. It is important to note that we also have external data available like CPI, Unemployment Rate and Fuel Prices in the region of each store which, hopefully, helps us to make a more detailed analysis. 
This data set is available on the kaggle website. These data sets contained information about the stores, departments, temperature, unemployment, CPI, isHoliday, and MarkDowns.

Stores :

Store: The store number. Range from 1–45.

Type: Three types of stores ‘A’, ‘B’ or ‘C’.

Size: Sets the size of a Store would be calculated by the no. of products available in the particular store ranging from 34,000 to 210,000.

Features:

Temperature: Temperature of the region during that week.

Fuel_Price: Fuel Price in that region during that week.

MarkDown1:5 : Represents the Type of markdown and what quantity was available 
during that week.

CPI: Consumer Price Index during that week.

Unemployment: The unemployment rate during that week in the region of the store.

Sales:

Date: The date of the week where this observation was taken.

Weekly_Sales: The sales recorded during that Week.

Dept: One of 1–99 that shows the department.

IsHoliday: a Boolean value representing a holiday week or not.

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

The 'Date' field doesn't represent the day itself, but each week, ending every friday. So, it's interesting two create a 'Week' field and also we can create one for 'Year'. 
```markdown
feat_store['Week'] = feat_store.Date.dt.week
feat_store['Year'] = feat_store.Date.dt.year
```
Merging the train and feat_store dataframe
```markdown
train_detail = train.merge(feat_store, on = ['Store','Date','IsHoliday']).sort_values(by = ['Store','Date','IsHoliday']).reset_index(drop=True)
train_detail
test_detail = test.merge(feat_store, on = ['Store', 'Date','IsHoliday']).sort_values(by = ['Store','Date','IsHoliday']).reset_index(drop = True)
test_detail
```
Let's take a look at the Average Weekly Sales per Year and find out if there are another holiday peak sales that were not considered by 'IsHoliday' field.
```markdown
weekly_sales_2010 = train_detail[train_detail.Year==2010]['Weekly_Sales'].groupby(train_detail['Week']).mean()
weekly_sales_2011 = train_detail[train_detail.Year==2011]['Weekly_Sales'].groupby(train_detail['Week']).mean()
weekly_sales_2012 = train_detail[train_detail.Year == 2012]['Weekly_Sales'].groupby(train_detail['Week']).mean()
fig = go.Figure()
fig.add_trace(go.Scatter(x = weekly_sales_2010.index, y = weekly_sales_2010.values, mode = 'lines', name = '2010_Weekly_Sales', line = dict(color = 'blue', width = 1.5)))
fig.add_trace(go.Scatter(x = weekly_sales_2011.index, y = weekly_sales_2011.values, mode = 'lines', name = '2011_Weekly_Sales', line = dict(color = 'green', width = 1.5)))
fig.add_trace(go.Scatter(x = weekly_sales_2012.index, y = weekly_sales_2012.values, mode = 'lines', name = '2012_Weekly_Sales', line = dict(color = 'maroon', width = 1.5)))
fig.update_layout(title = 'Average Weekly Sales Per Year')
fig.update_layout(xaxis = dict(tickmode = 'linear', tick0 = 1, dtick = 1))
fig.update_yaxes(title_text="Sales")
fig.update_xaxes(title_text = 'Week')
fig.show()
```
> As we can see, there is one important Holiday not included in 'IsHoliday'. It's the Easter Day. It is always in a Sunday, but can fall on different weeks. 
    >* In 2010 is in Week 13
    >* In 2011, Week 16
    >* Week 14 in 2012
    >* and, finally, Week 13 in 2013 for Test set

> So, we can change to 'True' these Weeks in each Year.
```markdown

train_detail.loc[(train_detail.Year == 2010) & (train_detail.Week == 13), 'IsHoliday'] = True
train_detail.loc[(train_detail.Year == 2011) & (train_detail.Week == 16), 'IsHoliday'] = True
train_detail.loc[(train_detail.Year == 2012) & (train_detail.Week == 14), "IsHoliday"] = True
test_detail.loc[(test_detail.Year == 2013) & (test_detail.Week == 13), 'IsHoliday'] = True
```
Now let's look the Averge weekly sales of each Store and identify the store having highest and lowest average weekly sales.
```markdown

weekly_sales = train_detail['Weekly_Sales'].groupby(train_detail['Store']).mean()
fig = px.bar(x = weekly_sales.index, y = weekly_sales.values, title = 'Averge Weekly Sales per store', color = weekly_sales.values)
fig.update_layout(title = 'Average Weekly Sales of Each Store')
fig.update_yaxes(title_text="Sales")
fig.update_xaxes(title_text = 'Store')
fig.show()
```
From above bar chart, we can identify the store having highest average weekly sales is store no. 20 with sales 29,508 and the store having lowest average weekly sales is store no. 5 with sales 5,053.




















