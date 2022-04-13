+++
author = "Justin Napolitano"
title = "European Natural Gas Imports by Country and Trading Partner"
date = "2022-04-13"
description = "Overview of European natural gas imports by country and trading partner."
tags = [
    "markdown",
    "css",
    "html",
    "python",
    "energy",
    "europe",
    "natural-gas"
]
categories = [
    "energy",
    "project",
    "python",
    "jupyterlab",
    "analysis"
]
series = ["European-Energy"]
images = ["/russian_natural_gas.png"]
aliases = ["european-gas-imports"]
+++

# Eurostat Natural Gas Data

## Import and Procedural Functions


```python
import pandas as pd
import matplotlib.pyplot as plt
import geopandas as gpd
import folium
import contextily as cx
import rtree
from zlib import crc32
import hashlib
from shapely.geometry import Point, LineString, Polygon
import numpy as np
from scipy.spatial import cKDTree
from shapely.geometry import Point
from haversine import Unit
from geopy.distance import distance
import re 
```

## Query Strategy

### Imports 

TO many to list... I'll ad them as I go below


## Glossary

### Units

Data should be reported in Terajoules (TJ) on the basis of Gross calorific values (GCV) and in million
cubic metres (at 15oC and 760 mm Hg, i.e. Standard Conditions) 

#### Source

https://ec.europa.eu/eurostat/documents/38154/10015688/Gas-Instructions-2018.pdf/2c78ed79-88fe-47ce-91f6-6069081755fb

## Data

## Converting all tsv to csv documents


### Building tsv file list


```python
import glob
 
# assign directory
directory = '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng'
 

tsv_list = [x for x in glob.iglob(f'{directory}/*.tsv')]

print(tsv_list)


```

    ['/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/imports_montth.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_imports.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_exports.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/supply_transformation_consumption.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/non_house_consumption.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_stocks.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/gas_prices.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/house_hold_consumption.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/exports_month.tsv', '/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/transport_costs.tsv']


### Converting to CSV

The columns of this data are ugly. Some are in csv format and others are in tab.  

I'll convert the entire document to csv to simplify our work.  


```python
for tsv_file in tsv_list:
  filepath = tsv_file
  outfile = filepath.replace('tsv','csv')
    
  # reading given tsv file 
  with open(filepath, 'r') as myfile:   
    with open(outfile, 'w') as csv_file: 
      for line in myfile: 
          
        # Replace every tab with comma 
        fileContent = re.sub("\t", ",", line) 
          
        # Writing into csv file 
        csv_file.write(fileContent) 
    
  # output 
  print(outfile)
```

    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/imports_montth.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_imports.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_exports.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/supply_transformation_consumption.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/non_house_consumption.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_stocks.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/gas_prices.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/house_hold_consumption.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/exports_month.csv
    /Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/transport_costs.csv


## Natural Gas Imports Data


```python
## Importing our DataFrames

filepath = "/Users/jnapolitano/Projects/data/EuropeanEnergyData/ng/ng_imports.csv"


## Reading the CSV Columns

ng_imports_df = pd.read_csv(filepath, skipinitialspace=True, low_memory=False)

# Repacing NA's
ng_imports_df = ng_imports_df.replace(to_replace=': ', value=0)

#coverting to float
ng_imports_df['1990 '] = ng_imports_df['1990 '].astype(float)
ng_imports_df['1991 '] = ng_imports_df['1991 '].astype(float)
ng_imports_df['1992 '] = ng_imports_df['1992 '].astype(float)
ng_imports_df['1993 '] = ng_imports_df['1993 '].astype(float)
ng_imports_df['1994 '] = ng_imports_df['1994 '].astype(float)
ng_imports_df['1995 '] = ng_imports_df['1995 '].astype(float)
ng_imports_df['1996 '] = ng_imports_df['1996 '].astype(float)
ng_imports_df['1997 '] = ng_imports_df['1997 '].astype(float)
ng_imports_df['1998 '] = ng_imports_df['1998 '].astype(float)
ng_imports_df['1999 '] = ng_imports_df['1999 '].astype(float)
ng_imports_df['2000 '] = ng_imports_df['2000 '].astype(float)
ng_imports_df['2001 '] = ng_imports_df['2001 '].astype(float)
ng_imports_df['2002 '] = ng_imports_df['2002 '].astype(float)
ng_imports_df['2003 '] = ng_imports_df['2003 '].astype(float)
ng_imports_df['2004 '] = ng_imports_df['2004 '].astype(float)
ng_imports_df['2005 '] = ng_imports_df['2005 '].astype(float)
ng_imports_df['2006 '] = ng_imports_df['2006 '].astype(float)
ng_imports_df['2007 '] = ng_imports_df['2007 '].astype(float)
ng_imports_df['2008 '] = ng_imports_df['2008 '].astype(float)
ng_imports_df['2009 '] = ng_imports_df['2009 '].astype(float)
ng_imports_df['2010 '] = ng_imports_df['2010 '].astype(float)
ng_imports_df['2011 '] = ng_imports_df['2011 '].astype(float)
ng_imports_df['2012 '] = ng_imports_df['2012 '].astype(float)
ng_imports_df['2013 '] = ng_imports_df['2013 '].astype(float)
ng_imports_df['2014 '] = ng_imports_df['2014 '].astype(float)
ng_imports_df['2015 '] = ng_imports_df['2015 '].astype(float)
ng_imports_df['2016 '] = ng_imports_df['2016 '].astype(float)
ng_imports_df['2017 '] = ng_imports_df['2017 '].astype(float)
ng_imports_df['2018 '] = ng_imports_df['2018 '].astype(float)
ng_imports_df['2019 '] = ng_imports_df['2019 '].astype(float)
ng_imports_df['2020 '] = ng_imports_df['2020 '].astype(float)

ng_imports_df.keys()

## This data set has spaces at the end of the years... Be aware of this when trying to findd keys

```




    Index(['siec', 'partner', 'unit', 'geo', '2020 ', '2019 ', '2018 ', '2017 ',
           '2016 ', '2015 ', '2014 ', '2013 ', '2012 ', '2011 ', '2010 ', '2009 ',
           '2008 ', '2007 ', '2006 ', '2005 ', '2004 ', '2003 ', '2002 ', '2001 ',
           '2000 ', '1999 ', '1998 ', '1997 ', '1996 ', '1995 ', '1994 ', '1993 ',
           '1992 ', '1991 ', '1990 '],
          dtype='object')



### Natural Gas Imports Descriptive Statistics


```python
ng_imports_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>2011</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>...</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
      <td>2.904000e+04</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>3.976261e+03</td>
      <td>6.114668e+03</td>
      <td>5.302002e+03</td>
      <td>5.539567e+03</td>
      <td>4.955097e+03</td>
      <td>4.836956e+03</td>
      <td>4.542184e+03</td>
      <td>4.855483e+03</td>
      <td>5.003112e+03</td>
      <td>5.339544e+03</td>
      <td>...</td>
      <td>3.360443e+03</td>
      <td>3.063990e+03</td>
      <td>3.041415e+03</td>
      <td>2.983608e+03</td>
      <td>2.688182e+03</td>
      <td>2.459352e+03</td>
      <td>2.457050e+03</td>
      <td>2.535403e+03</td>
      <td>2.519013e+03</td>
      <td>2.476906e+03</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1.390934e+05</td>
      <td>1.951595e+05</td>
      <td>1.789466e+05</td>
      <td>1.908091e+05</td>
      <td>1.727564e+05</td>
      <td>1.664602e+05</td>
      <td>1.551553e+05</td>
      <td>1.670085e+05</td>
      <td>1.656728e+05</td>
      <td>1.711071e+05</td>
      <td>...</td>
      <td>1.201655e+05</td>
      <td>1.106808e+05</td>
      <td>1.080009e+05</td>
      <td>1.080827e+05</td>
      <td>9.770677e+04</td>
      <td>8.894334e+04</td>
      <td>8.770472e+04</td>
      <td>9.028499e+04</td>
      <td>9.125984e+04</td>
      <td>9.071060e+04</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.531362e+07</td>
      <td>1.862959e+07</td>
      <td>1.720083e+07</td>
      <td>1.823810e+07</td>
      <td>1.662462e+07</td>
      <td>1.602176e+07</td>
      <td>1.501862e+07</td>
      <td>1.617363e+07</td>
      <td>1.615217e+07</td>
      <td>1.666649e+07</td>
      <td>...</td>
      <td>1.049217e+07</td>
      <td>9.634890e+06</td>
      <td>9.420641e+06</td>
      <td>9.378006e+06</td>
      <td>8.381725e+06</td>
      <td>7.600521e+06</td>
      <td>7.500412e+06</td>
      <td>7.641602e+06</td>
      <td>7.664437e+06</td>
      <td>7.627579e+06</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 31 columns</p>
</div>



### Replace 0's with Numpy NAN


```python
ng_imports_df.replace(0, np.nan, inplace=True)

```

### Remove NAN Rows for the dataframe

I remove rows where all of the rows are 0 to reduce the size of the dataframe for analysis.


```python
## There are 30 years of data in the set. If there are 30 columns per row with nan values that row will be dropped.
ng_imports_df.dropna(thresh=30, axis=0, inplace=True)
#df.dropna(thresh=2)
```


```python
ng_imports_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>2011</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1.860000e+02</td>
      <td>2.140000e+02</td>
      <td>2.140000e+02</td>
      <td>2.140000e+02</td>
      <td>2.140000e+02</td>
      <td>2.160000e+02</td>
      <td>2.160000e+02</td>
      <td>2.160000e+02</td>
      <td>2.160000e+02</td>
      <td>2.160000e+02</td>
      <td>...</td>
      <td>2.160000e+02</td>
      <td>2.140000e+02</td>
      <td>2.140000e+02</td>
      <td>2.100000e+02</td>
      <td>2.120000e+02</td>
      <td>2.160000e+02</td>
      <td>2.100000e+02</td>
      <td>1.900000e+02</td>
      <td>1.840000e+02</td>
      <td>1.840000e+02</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.054436e+05</td>
      <td>6.803278e+05</td>
      <td>6.255655e+05</td>
      <td>6.649271e+05</td>
      <td>6.019034e+05</td>
      <td>5.789388e+05</td>
      <td>5.452416e+05</td>
      <td>5.802616e+05</td>
      <td>5.794676e+05</td>
      <td>6.005061e+05</td>
      <td>...</td>
      <td>4.287046e+05</td>
      <td>3.973935e+05</td>
      <td>3.920810e+05</td>
      <td>3.919102e+05</td>
      <td>3.495936e+05</td>
      <td>3.136714e+05</td>
      <td>3.210325e+05</td>
      <td>3.637285e+05</td>
      <td>3.731901e+05</td>
      <td>3.674058e+05</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1.657836e+06</td>
      <td>2.161218e+06</td>
      <td>1.987492e+06</td>
      <td>2.120905e+06</td>
      <td>1.920621e+06</td>
      <td>1.840135e+06</td>
      <td>1.713982e+06</td>
      <td>1.846429e+06</td>
      <td>1.826193e+06</td>
      <td>1.877469e+06</td>
      <td>...</td>
      <td>1.321855e+06</td>
      <td>1.221276e+06</td>
      <td>1.190700e+06</td>
      <td>1.202703e+06</td>
      <td>1.082313e+06</td>
      <td>9.743030e+05</td>
      <td>9.710055e+05</td>
      <td>1.036258e+06</td>
      <td>1.056681e+06</td>
      <td>1.051903e+06</td>
    </tr>
    <tr>
      <th>min</th>
      <td>7.804500e+01</td>
      <td>1.057770e+02</td>
      <td>2.766250e+02</td>
      <td>2.075750e+02</td>
      <td>8.588100e+01</td>
      <td>5.400000e+01</td>
      <td>1.047720e+02</td>
      <td>1.111500e+02</td>
      <td>1.288710e+02</td>
      <td>1.247130e+02</td>
      <td>...</td>
      <td>3.850000e+02</td>
      <td>3.340000e+02</td>
      <td>1.960000e+02</td>
      <td>7.900000e+01</td>
      <td>1.600000e+01</td>
      <td>3.000000e+00</td>
      <td>5.000000e+00</td>
      <td>4.000000e+00</td>
      <td>1.400000e+02</td>
      <td>2.400000e+01</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>7.764267e+03</td>
      <td>9.640211e+03</td>
      <td>9.129516e+03</td>
      <td>9.197332e+03</td>
      <td>8.621260e+03</td>
      <td>7.365000e+03</td>
      <td>7.272250e+03</td>
      <td>8.467000e+03</td>
      <td>8.132250e+03</td>
      <td>8.680500e+03</td>
      <td>...</td>
      <td>6.344250e+03</td>
      <td>5.477500e+03</td>
      <td>5.135750e+03</td>
      <td>5.386000e+03</td>
      <td>3.794750e+03</td>
      <td>3.599000e+03</td>
      <td>4.431250e+03</td>
      <td>5.417500e+03</td>
      <td>5.554000e+03</td>
      <td>5.988000e+03</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.631806e+04</td>
      <td>6.532805e+04</td>
      <td>5.295587e+04</td>
      <td>5.474258e+04</td>
      <td>4.763608e+04</td>
      <td>4.939550e+04</td>
      <td>4.918376e+04</td>
      <td>5.242819e+04</td>
      <td>5.133605e+04</td>
      <td>5.896975e+04</td>
      <td>...</td>
      <td>4.032900e+04</td>
      <td>3.802100e+04</td>
      <td>3.999400e+04</td>
      <td>3.692250e+04</td>
      <td>3.121600e+04</td>
      <td>2.416850e+04</td>
      <td>2.362300e+04</td>
      <td>3.236600e+04</td>
      <td>3.257000e+04</td>
      <td>3.202500e+04</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>3.430158e+05</td>
      <td>4.126836e+05</td>
      <td>3.347255e+05</td>
      <td>3.842234e+05</td>
      <td>3.636243e+05</td>
      <td>2.981946e+05</td>
      <td>3.150444e+05</td>
      <td>3.513390e+05</td>
      <td>3.349404e+05</td>
      <td>3.565156e+05</td>
      <td>...</td>
      <td>2.399850e+05</td>
      <td>2.132655e+05</td>
      <td>2.171552e+05</td>
      <td>2.316358e+05</td>
      <td>2.124592e+05</td>
      <td>1.776518e+05</td>
      <td>1.811268e+05</td>
      <td>2.042445e+05</td>
      <td>2.132402e+05</td>
      <td>2.414440e+05</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.531362e+07</td>
      <td>1.862959e+07</td>
      <td>1.720083e+07</td>
      <td>1.823810e+07</td>
      <td>1.662462e+07</td>
      <td>1.602176e+07</td>
      <td>1.501862e+07</td>
      <td>1.617363e+07</td>
      <td>1.615217e+07</td>
      <td>1.666649e+07</td>
      <td>...</td>
      <td>1.049217e+07</td>
      <td>9.634890e+06</td>
      <td>9.420641e+06</td>
      <td>9.378006e+06</td>
      <td>8.381725e+06</td>
      <td>7.600521e+06</td>
      <td>7.500412e+06</td>
      <td>7.641602e+06</td>
      <td>7.664437e+06</td>
      <td>7.627579e+06</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 31 columns</p>
</div>



### Grouping Totals by Geo/State


```python
geo_imports_df = ng_imports_df.groupby(['geo', 'partner','unit']).sum()

```


```python
geo_imports_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>2011</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
    <tr>
      <th>geo</th>
      <th>partner</th>
      <th>unit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">AT</th>
      <th rowspan="2" valign="top">NSP</th>
      <th>MIO_M3</th>
      <td>16455.858</td>
      <td>14191.254</td>
      <td>13055.236</td>
      <td>13923.121</td>
      <td>14434.033</td>
      <td>11890.337</td>
      <td>10119.522</td>
      <td>2365.164</td>
      <td>3240.632</td>
      <td>2954.378</td>
      <td>...</td>
      <td>385.0</td>
      <td>334.0</td>
      <td>318.0</td>
      <td>496.0</td>
      <td>313.0</td>
      <td>249.0</td>
      <td>246.0</td>
      <td>241.0</td>
      <td>240.0</td>
      <td>147.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>636210.000</td>
      <td>547204.000</td>
      <td>503400.001</td>
      <td>536865.000</td>
      <td>557057.893</td>
      <td>456859.409</td>
      <td>388129.243</td>
      <td>92877.999</td>
      <td>126750.998</td>
      <td>115683.999</td>
      <td>...</td>
      <td>16813.0</td>
      <td>14961.0</td>
      <td>14289.0</td>
      <td>21281.0</td>
      <td>14307.0</td>
      <td>11340.0</td>
      <td>11378.0</td>
      <td>11049.0</td>
      <td>11050.0</td>
      <td>7647.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">TOTAL</th>
      <th>MIO_M3</th>
      <td>16455.858</td>
      <td>14191.254</td>
      <td>13055.236</td>
      <td>13923.121</td>
      <td>14434.033</td>
      <td>11890.337</td>
      <td>10119.522</td>
      <td>10379.164</td>
      <td>14171.632</td>
      <td>13381.378</td>
      <td>...</td>
      <td>6463.0</td>
      <td>6565.0</td>
      <td>6357.0</td>
      <td>6933.0</td>
      <td>6714.0</td>
      <td>5258.0</td>
      <td>5676.0</td>
      <td>5388.0</td>
      <td>5396.0</td>
      <td>5507.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>636210.000</td>
      <td>547204.000</td>
      <td>503400.001</td>
      <td>536865.000</td>
      <td>557057.893</td>
      <td>456859.409</td>
      <td>388129.243</td>
      <td>396670.999</td>
      <td>541127.998</td>
      <td>510952.999</td>
      <td>...</td>
      <td>243872.0</td>
      <td>248898.0</td>
      <td>241012.0</td>
      <td>262866.0</td>
      <td>254571.0</td>
      <td>199367.0</td>
      <td>215218.0</td>
      <td>204274.0</td>
      <td>204599.0</td>
      <td>208797.0</td>
    </tr>
    <tr>
      <th>BE</th>
      <th>NL</th>
      <th>MIO_M3</th>
      <td>7758.900</td>
      <td>8153.600</td>
      <td>9022.600</td>
      <td>9214.100</td>
      <td>8262.600</td>
      <td>7552.900</td>
      <td>8070.900</td>
      <td>8749.000</td>
      <td>7211.000</td>
      <td>7250.000</td>
      <td>...</td>
      <td>5715.0</td>
      <td>5635.0</td>
      <td>4937.0</td>
      <td>6055.0</td>
      <td>4925.0</td>
      <td>4942.0</td>
      <td>4610.0</td>
      <td>4288.0</td>
      <td>3997.0</td>
      <td>3945.0</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>UA</th>
      <th>TOTAL</th>
      <th>TJ_GCV</th>
      <td>343593.000</td>
      <td>442224.820</td>
      <td>393516.000</td>
      <td>523927.000</td>
      <td>409787.000</td>
      <td>618333.000</td>
      <td>731493.000</td>
      <td>1051134.000</td>
      <td>1237329.000</td>
      <td>1683533.000</td>
      <td>...</td>
      <td>2338789.0</td>
      <td>2089490.0</td>
      <td>2433127.0</td>
      <td>2778224.0</td>
      <td>2477224.0</td>
      <td>2494496.0</td>
      <td>2887820.0</td>
      <td>3377128.0</td>
      <td>3457090.0</td>
      <td>3418435.0</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">UK</th>
      <th rowspan="2" valign="top">NO</th>
      <th>MIO_M3</th>
      <td>0.000</td>
      <td>27321.556</td>
      <td>34385.304</td>
      <td>35889.765</td>
      <td>31966.853</td>
      <td>28159.067</td>
      <td>25561.000</td>
      <td>29084.000</td>
      <td>28448.000</td>
      <td>23442.000</td>
      <td>...</td>
      <td>598.0</td>
      <td>815.0</td>
      <td>1272.0</td>
      <td>1716.0</td>
      <td>1715.0</td>
      <td>3026.0</td>
      <td>4474.0</td>
      <td>5649.0</td>
      <td>6358.0</td>
      <td>7175.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>0.000</td>
      <td>1078490.545</td>
      <td>1355791.847</td>
      <td>1416302.465</td>
      <td>1259353.868</td>
      <td>1110757.211</td>
      <td>1003743.000</td>
      <td>1150926.000</td>
      <td>1128495.000</td>
      <td>935124.000</td>
      <td>...</td>
      <td>25266.0</td>
      <td>33748.0</td>
      <td>50623.0</td>
      <td>71295.0</td>
      <td>70045.0</td>
      <td>118991.0</td>
      <td>174702.0</td>
      <td>220519.0</td>
      <td>259228.0</td>
      <td>285468.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">TOTAL</th>
      <th>MIO_M3</th>
      <td>0.000</td>
      <td>47380.347</td>
      <td>47160.385</td>
      <td>47155.774</td>
      <td>48243.028</td>
      <td>45276.153</td>
      <td>43820.000</td>
      <td>49309.000</td>
      <td>50716.000</td>
      <td>54622.000</td>
      <td>...</td>
      <td>1135.0</td>
      <td>929.0</td>
      <td>1272.0</td>
      <td>1716.0</td>
      <td>1715.0</td>
      <td>3026.0</td>
      <td>4474.0</td>
      <td>5649.0</td>
      <td>6358.0</td>
      <td>7226.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>0.000</td>
      <td>1864485.126</td>
      <td>1861489.821</td>
      <td>1861271.973</td>
      <td>1896723.089</td>
      <td>1783416.506</td>
      <td>1717087.000</td>
      <td>1937305.000</td>
      <td>1995680.000</td>
      <td>2150889.000</td>
      <td>...</td>
      <td>46303.0</td>
      <td>38095.0</td>
      <td>50623.0</td>
      <td>71295.0</td>
      <td>70045.0</td>
      <td>118991.0</td>
      <td>174702.0</td>
      <td>220519.0</td>
      <td>259228.0</td>
      <td>287407.0</td>
    </tr>
  </tbody>
</table>
<p>186 rows × 31 columns</p>
</div>



With the tables compiled we can now look more in depthly at the imports of natural gas in europe.  


## State Code Reference Sheet Source 

https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Glossary:Country_codes

### German Totals 


```python
german_totals_df = geo_imports_df.query('geo=="DE"')
#geo_imports_df.loc[pd.IndexSlice[:,'DE'],:]

```

German Totals Bar Chart


```python
german_totals_df.plot(legend=True, figsize=(15,15), kind = 'bar')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_28_1.png)
    


### German Totals Line

Only including long term partners


```python
german_totals_df.plot(legend=True, figsize=(15,15), kind = 'line')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_30_1.png)
    


### French Totals 


```python
french_totals_df=geo_imports_df.query('geo=="FR"')
french_totals_df
#geo_imports_df.loc[pd.IndexSlice[:,'DE'],:]

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>2011</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
    <tr>
      <th>geo</th>
      <th>partner</th>
      <th>unit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">FR</th>
      <th rowspan="2" valign="top">DZ</th>
      <th>MIO_M3</th>
      <td>7246.028</td>
      <td>7523.508</td>
      <td>6894.430</td>
      <td>7555.354</td>
      <td>9409.600</td>
      <td>7681.532</td>
      <td>8356.126</td>
      <td>9674.938</td>
      <td>7678.870</td>
      <td>11100.376</td>
      <td>...</td>
      <td>19688.0</td>
      <td>19028.0</td>
      <td>18880.0</td>
      <td>14994.0</td>
      <td>14206.0</td>
      <td>15146.0</td>
      <td>17030.0</td>
      <td>17592.0</td>
      <td>17240.0</td>
      <td>17424.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>302594.104</td>
      <td>314181.712</td>
      <td>287911.434</td>
      <td>315511.606</td>
      <td>392944.880</td>
      <td>320780.758</td>
      <td>348951.826</td>
      <td>404025.436</td>
      <td>320669.632</td>
      <td>463551.722</td>
      <td>...</td>
      <td>829282.0</td>
      <td>801446.0</td>
      <td>768024.0</td>
      <td>609926.0</td>
      <td>608544.0</td>
      <td>648822.0</td>
      <td>729914.0</td>
      <td>758722.0</td>
      <td>743558.0</td>
      <td>751672.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NL</th>
      <th>MIO_M3</th>
      <td>3861.433</td>
      <td>4542.714</td>
      <td>5303.961</td>
      <td>5454.700</td>
      <td>5818.945</td>
      <td>5969.006</td>
      <td>6400.357</td>
      <td>8148.388</td>
      <td>8441.102</td>
      <td>9601.455</td>
      <td>...</td>
      <td>5601.0</td>
      <td>5425.0</td>
      <td>5075.0</td>
      <td>5810.0</td>
      <td>4975.0</td>
      <td>4658.0</td>
      <td>4520.0</td>
      <td>5506.0</td>
      <td>5003.0</td>
      <td>4111.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>144572.041</td>
      <td>170079.226</td>
      <td>198580.312</td>
      <td>204223.967</td>
      <td>217861.295</td>
      <td>223479.578</td>
      <td>239629.352</td>
      <td>305075.646</td>
      <td>316034.863</td>
      <td>359478.479</td>
      <td>...</td>
      <td>205682.0</td>
      <td>199202.0</td>
      <td>188168.0</td>
      <td>215435.0</td>
      <td>182859.0</td>
      <td>172706.0</td>
      <td>167601.0</td>
      <td>204156.0</td>
      <td>185504.0</td>
      <td>152374.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NO</th>
      <th>MIO_M3</th>
      <td>16652.066</td>
      <td>19478.315</td>
      <td>18895.797</td>
      <td>20245.741</td>
      <td>19595.158</td>
      <td>21014.676</td>
      <td>20369.749</td>
      <td>19022.821</td>
      <td>20003.327</td>
      <td>17014.616</td>
      <td>...</td>
      <td>12839.0</td>
      <td>10231.0</td>
      <td>10590.0</td>
      <td>10224.0</td>
      <td>7701.0</td>
      <td>7304.0</td>
      <td>5848.0</td>
      <td>6070.0</td>
      <td>5776.0</td>
      <td>5483.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>695390.296</td>
      <td>813414.448</td>
      <td>789088.464</td>
      <td>845462.132</td>
      <td>818293.792</td>
      <td>877572.888</td>
      <td>850640.736</td>
      <td>794393.008</td>
      <td>835338.919</td>
      <td>710530.350</td>
      <td>...</td>
      <td>536137.0</td>
      <td>433991.0</td>
      <td>442224.0</td>
      <td>426974.0</td>
      <td>318827.0</td>
      <td>302378.0</td>
      <td>242122.0</td>
      <td>251291.0</td>
      <td>239130.0</td>
      <td>227221.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">RU</th>
      <th>MIO_M3</th>
      <td>7780.367</td>
      <td>10772.464</td>
      <td>9812.886</td>
      <td>9075.273</td>
      <td>9391.739</td>
      <td>5694.684</td>
      <td>6411.893</td>
      <td>9410.315</td>
      <td>7507.628</td>
      <td>6755.661</td>
      <td>...</td>
      <td>12068.0</td>
      <td>10022.0</td>
      <td>10228.0</td>
      <td>11573.0</td>
      <td>12165.0</td>
      <td>11375.0</td>
      <td>10757.0</td>
      <td>11268.0</td>
      <td>10639.0</td>
      <td>9891.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>324908.143</td>
      <td>449858.107</td>
      <td>409786.128</td>
      <td>378983.407</td>
      <td>392199.021</td>
      <td>237809.998</td>
      <td>267760.652</td>
      <td>392974.766</td>
      <td>313518.525</td>
      <td>282116.424</td>
      <td>...</td>
      <td>482256.0</td>
      <td>400479.0</td>
      <td>405029.0</td>
      <td>458287.0</td>
      <td>481748.0</td>
      <td>450443.0</td>
      <td>425974.0</td>
      <td>446260.0</td>
      <td>421312.0</td>
      <td>392123.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">TOTAL</th>
      <th>MIO_M3</th>
      <td>63289.839</td>
      <td>75264.422</td>
      <td>59900.431</td>
      <td>57830.737</td>
      <td>53386.198</td>
      <td>50021.714</td>
      <td>51489.814</td>
      <td>55647.460</td>
      <td>57271.263</td>
      <td>63714.378</td>
      <td>...</td>
      <td>50469.0</td>
      <td>44706.0</td>
      <td>44773.0</td>
      <td>42715.0</td>
      <td>39542.0</td>
      <td>38483.0</td>
      <td>38155.0</td>
      <td>40436.0</td>
      <td>38658.0</td>
      <td>36909.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>2626302.288</td>
      <td>3123417.779</td>
      <td>2478528.880</td>
      <td>2391447.319</td>
      <td>2204269.799</td>
      <td>2063120.688</td>
      <td>2122565.095</td>
      <td>2288636.915</td>
      <td>2355182.424</td>
      <td>2619234.164</td>
      <td>...</td>
      <td>2064351.0</td>
      <td>1835118.0</td>
      <td>1803445.0</td>
      <td>1715144.0</td>
      <td>1611918.0</td>
      <td>1574349.0</td>
      <td>1565611.0</td>
      <td>1660429.0</td>
      <td>1589504.0</td>
      <td>1523390.0</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 31 columns</p>
</div>



### French Totals Bar Chart


```python
french_totals_df.plot(legend=True, figsize=(20,15), kind = 'bar')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_34_1.png)
    


### French Totals Line

Only long term partners


```python
french_totals_df.plot(legend=True, figsize=(15,15), kind = 'line')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_36_1.png)
    


## Max Importers Per Year


```python
max_imports_df = ng_imports_df.groupby(['geo', 'partner','unit']).max()
```

### French Maximums


```python
french_max_df=max_imports_df.query('geo=="FR"')
french_max_df

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>siec</th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
    <tr>
      <th>geo</th>
      <th>partner</th>
      <th>unit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">FR</th>
      <th rowspan="2" valign="top">DZ</th>
      <th>MIO_M3</th>
      <td>G3200</td>
      <td>3623.014</td>
      <td>3761.754</td>
      <td>3447.215</td>
      <td>3777.677</td>
      <td>4704.800</td>
      <td>3840.766</td>
      <td>4178.063</td>
      <td>4837.469</td>
      <td>3839.435</td>
      <td>...</td>
      <td>9844.0</td>
      <td>9514.0</td>
      <td>9440.0</td>
      <td>7497.0</td>
      <td>7103.0</td>
      <td>7573.0</td>
      <td>8515.0</td>
      <td>8796.0</td>
      <td>8620.0</td>
      <td>8712.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3200</td>
      <td>151297.052</td>
      <td>157090.856</td>
      <td>143955.717</td>
      <td>157755.803</td>
      <td>196472.440</td>
      <td>160390.379</td>
      <td>174475.913</td>
      <td>202012.718</td>
      <td>160334.816</td>
      <td>...</td>
      <td>414641.0</td>
      <td>400723.0</td>
      <td>384012.0</td>
      <td>304963.0</td>
      <td>304272.0</td>
      <td>324411.0</td>
      <td>364957.0</td>
      <td>379361.0</td>
      <td>371779.0</td>
      <td>375836.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NL</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>3861.433</td>
      <td>4542.714</td>
      <td>5303.961</td>
      <td>5454.700</td>
      <td>5818.945</td>
      <td>5969.006</td>
      <td>6400.357</td>
      <td>8148.388</td>
      <td>8441.102</td>
      <td>...</td>
      <td>5601.0</td>
      <td>5425.0</td>
      <td>5075.0</td>
      <td>5810.0</td>
      <td>4975.0</td>
      <td>4658.0</td>
      <td>4520.0</td>
      <td>5506.0</td>
      <td>5003.0</td>
      <td>4111.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>144572.041</td>
      <td>170079.226</td>
      <td>198580.312</td>
      <td>204223.967</td>
      <td>217861.295</td>
      <td>223479.578</td>
      <td>239629.352</td>
      <td>305075.646</td>
      <td>316034.863</td>
      <td>...</td>
      <td>205682.0</td>
      <td>199202.0</td>
      <td>188168.0</td>
      <td>215435.0</td>
      <td>182859.0</td>
      <td>172706.0</td>
      <td>167601.0</td>
      <td>204156.0</td>
      <td>185504.0</td>
      <td>152374.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NO</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>16652.066</td>
      <td>19478.315</td>
      <td>18895.797</td>
      <td>20245.741</td>
      <td>19595.158</td>
      <td>21014.676</td>
      <td>20369.749</td>
      <td>19022.821</td>
      <td>20003.327</td>
      <td>...</td>
      <td>12839.0</td>
      <td>10231.0</td>
      <td>10590.0</td>
      <td>10224.0</td>
      <td>7701.0</td>
      <td>7304.0</td>
      <td>5848.0</td>
      <td>6070.0</td>
      <td>5776.0</td>
      <td>5483.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>695390.296</td>
      <td>813414.448</td>
      <td>789088.464</td>
      <td>845462.132</td>
      <td>818293.792</td>
      <td>877572.888</td>
      <td>850640.736</td>
      <td>794393.008</td>
      <td>835338.919</td>
      <td>...</td>
      <td>536137.0</td>
      <td>433991.0</td>
      <td>442224.0</td>
      <td>426974.0</td>
      <td>318827.0</td>
      <td>302378.0</td>
      <td>242122.0</td>
      <td>251291.0</td>
      <td>239130.0</td>
      <td>227221.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">RU</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>7780.367</td>
      <td>10772.464</td>
      <td>9812.886</td>
      <td>9075.273</td>
      <td>9391.739</td>
      <td>5694.684</td>
      <td>6411.893</td>
      <td>9410.315</td>
      <td>7507.628</td>
      <td>...</td>
      <td>12068.0</td>
      <td>10022.0</td>
      <td>10228.0</td>
      <td>11573.0</td>
      <td>12165.0</td>
      <td>11375.0</td>
      <td>10757.0</td>
      <td>11268.0</td>
      <td>10639.0</td>
      <td>9891.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>324908.143</td>
      <td>449858.107</td>
      <td>409786.128</td>
      <td>378983.407</td>
      <td>392199.021</td>
      <td>237809.998</td>
      <td>267760.652</td>
      <td>392974.766</td>
      <td>313518.525</td>
      <td>...</td>
      <td>482256.0</td>
      <td>400479.0</td>
      <td>405029.0</td>
      <td>458287.0</td>
      <td>481748.0</td>
      <td>450443.0</td>
      <td>425974.0</td>
      <td>446260.0</td>
      <td>421312.0</td>
      <td>392123.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">TOTAL</th>
      <th>MIO_M3</th>
      <td>G3200</td>
      <td>46321.129</td>
      <td>54948.085</td>
      <td>49442.320</td>
      <td>48638.995</td>
      <td>46536.484</td>
      <td>44487.899</td>
      <td>45436.533</td>
      <td>48205.567</td>
      <td>48060.439</td>
      <td>...</td>
      <td>40625.0</td>
      <td>35192.0</td>
      <td>35333.0</td>
      <td>35218.0</td>
      <td>32439.0</td>
      <td>30910.0</td>
      <td>29640.0</td>
      <td>31640.0</td>
      <td>30038.0</td>
      <td>28197.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3200</td>
      <td>1917688.977</td>
      <td>2275007.484</td>
      <td>2041798.166</td>
      <td>2007600.132</td>
      <td>1918225.733</td>
      <td>1832028.575</td>
      <td>1869780.080</td>
      <td>1977863.469</td>
      <td>1970538.372</td>
      <td>...</td>
      <td>1649710.0</td>
      <td>1434395.0</td>
      <td>1419433.0</td>
      <td>1410181.0</td>
      <td>1307646.0</td>
      <td>1249938.0</td>
      <td>1200654.0</td>
      <td>1281068.0</td>
      <td>1217725.0</td>
      <td>1147554.0</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 32 columns</p>
</div>



### French Max Bar Chart


```python
french_max_df.plot(legend=True, figsize=(15,15),kind = 'bar')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_42_1.png)
    


### French Max Line
** Only long term partners


```python
french_max_df.plot(legend=True, figsize=(15,15))

```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_44_1.png)
    


## German Maximums


```python
german_max_df = max_imports_df.query('geo=="DE"')
german_max_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>siec</th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
    <tr>
      <th>geo</th>
      <th>partner</th>
      <th>unit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">DE</th>
      <th rowspan="2" valign="top">NL</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>10211.684</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>33863.0</td>
      <td>28053.0</td>
      <td>30971.0</td>
      <td>25952.0</td>
      <td>...</td>
      <td>21163.0</td>
      <td>22417.0</td>
      <td>24983.0</td>
      <td>29933.0</td>
      <td>25254.0</td>
      <td>23832.0</td>
      <td>26527.0</td>
      <td>25064.0</td>
      <td>23713.0</td>
      <td>19591.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>359134.724</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1320260.0</td>
      <td>1093730.0</td>
      <td>1207497.0</td>
      <td>1011835.0</td>
      <td>...</td>
      <td>704733.0</td>
      <td>746474.0</td>
      <td>831931.0</td>
      <td>996760.0</td>
      <td>840951.0</td>
      <td>793608.0</td>
      <td>883342.0</td>
      <td>834641.0</td>
      <td>789635.0</td>
      <td>652396.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NO</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>16484.581</td>
      <td>2639.601</td>
      <td>2460.283</td>
      <td>11111.404</td>
      <td>10705.0</td>
      <td>21994.0</td>
      <td>20801.0</td>
      <td>20257.0</td>
      <td>24482.0</td>
      <td>...</td>
      <td>17440.0</td>
      <td>16838.0</td>
      <td>18279.0</td>
      <td>15309.0</td>
      <td>11369.0</td>
      <td>10287.0</td>
      <td>9519.0</td>
      <td>9380.0</td>
      <td>8255.0</td>
      <td>8140.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>642700.831</td>
      <td>102912.771</td>
      <td>95921.526</td>
      <td>433211.418</td>
      <td>417362.0</td>
      <td>857516.0</td>
      <td>811005.0</td>
      <td>789784.0</td>
      <td>954485.0</td>
      <td>...</td>
      <td>720273.0</td>
      <td>695423.0</td>
      <td>754943.0</td>
      <td>632264.0</td>
      <td>469546.0</td>
      <td>424842.0</td>
      <td>393148.0</td>
      <td>387390.0</td>
      <td>340911.0</td>
      <td>336170.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NSP</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>1279.189</td>
      <td>45897.173</td>
      <td>42779.209</td>
      <td>45503.256</td>
      <td>28003.0</td>
      <td>3034.0</td>
      <td>3841.0</td>
      <td>6572.0</td>
      <td>5335.0</td>
      <td>...</td>
      <td>3744.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>29.0</td>
      <td>28.0</td>
      <td>18.0</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>44987.785</td>
      <td>1614157.693</td>
      <td>1504501.985</td>
      <td>1600304.025</td>
      <td>1091777.0</td>
      <td>118283.0</td>
      <td>149763.0</td>
      <td>256226.0</td>
      <td>208000.0</td>
      <td>...</td>
      <td>149721.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>971.0</td>
      <td>940.0</td>
      <td>615.0</td>
      <td>121.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">RU</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>52463.565</td>
      <td>46249.986</td>
      <td>43108.053</td>
      <td>62079.783</td>
      <td>58671.0</td>
      <td>43626.0</td>
      <td>37201.0</td>
      <td>39977.0</td>
      <td>32632.0</td>
      <td>...</td>
      <td>34414.0</td>
      <td>32673.0</td>
      <td>30659.0</td>
      <td>32444.0</td>
      <td>32134.0</td>
      <td>29195.0</td>
      <td>25201.0</td>
      <td>22294.0</td>
      <td>23969.0</td>
      <td>26160.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>2045449.486</td>
      <td>1803194.439</td>
      <td>1680696.766</td>
      <td>2420366.562</td>
      <td>2287481.0</td>
      <td>1700891.0</td>
      <td>1450379.0</td>
      <td>1558631.0</td>
      <td>1272265.0</td>
      <td>...</td>
      <td>1290507.0</td>
      <td>1225235.0</td>
      <td>1149679.0</td>
      <td>1217704.0</td>
      <td>1205187.0</td>
      <td>1095350.0</td>
      <td>945645.0</td>
      <td>838330.0</td>
      <td>899264.0</td>
      <td>981006.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">TOTAL</th>
      <th>MIO_M3</th>
      <td>G3000</td>
      <td>80439.019</td>
      <td>94786.760</td>
      <td>88347.545</td>
      <td>118694.443</td>
      <td>97379.0</td>
      <td>102517.0</td>
      <td>89896.0</td>
      <td>97777.0</td>
      <td>88401.0</td>
      <td>...</td>
      <td>76761.0</td>
      <td>74247.0</td>
      <td>76594.0</td>
      <td>79555.0</td>
      <td>70208.0</td>
      <td>64889.0</td>
      <td>62343.0</td>
      <td>58229.0</td>
      <td>56789.0</td>
      <td>54288.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>G3000</td>
      <td>3092272.826</td>
      <td>3520264.903</td>
      <td>3281120.277</td>
      <td>4453882.005</td>
      <td>3796620.0</td>
      <td>3996950.0</td>
      <td>3504877.0</td>
      <td>3812138.0</td>
      <td>3446585.0</td>
      <td>...</td>
      <td>2865234.0</td>
      <td>2760192.0</td>
      <td>2845235.0</td>
      <td>2919984.0</td>
      <td>2573617.0</td>
      <td>2375961.0</td>
      <td>2265243.0</td>
      <td>2115799.0</td>
      <td>2063657.0</td>
      <td>1985817.0</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 32 columns</p>
</div>



### German Max Bar Chart


```python
german_max_df.plot(legend=True, figsize=(15,15),kind = 'bar')
```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_48_1.png)
    


### German Max Line
** Only long term partners


```python
german_max_df.plot(legend=True, figsize=(15,15))

```




    <AxesSubplot:xlabel='geo,partner,unit'>




    
![png](output_50_1.png)
    


## Gas Partners


```python
partners_sum_df = ng_imports_df.groupby(['partner', 'geo','unit']).sum()
```


```python
partners_sum_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>2020</th>
      <th>2019</th>
      <th>2018</th>
      <th>2017</th>
      <th>2016</th>
      <th>2015</th>
      <th>2014</th>
      <th>2013</th>
      <th>2012</th>
      <th>2011</th>
      <th>...</th>
      <th>1999</th>
      <th>1998</th>
      <th>1997</th>
      <th>1996</th>
      <th>1995</th>
      <th>1994</th>
      <th>1993</th>
      <th>1992</th>
      <th>1991</th>
      <th>1990</th>
    </tr>
    <tr>
      <th>partner</th>
      <th>geo</th>
      <th>unit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">DE</th>
      <th rowspan="2" valign="top">EU27_2020</th>
      <th>MIO_M3</th>
      <td>9439.349</td>
      <td>7861.448</td>
      <td>7198.430</td>
      <td>10861.035</td>
      <td>8628.038</td>
      <td>7959.978</td>
      <td>8282.820</td>
      <td>11233.747</td>
      <td>9064.757</td>
      <td>7553.156</td>
      <td>...</td>
      <td>1348.0</td>
      <td>1690.0</td>
      <td>827.0</td>
      <td>102.0</td>
      <td>39.0</td>
      <td>33.0</td>
      <td>21.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>348817.511</td>
      <td>293747.051</td>
      <td>260201.122</td>
      <td>385886.897</td>
      <td>320281.116</td>
      <td>298190.008</td>
      <td>301892.481</td>
      <td>410193.681</td>
      <td>334940.412</td>
      <td>285260.821</td>
      <td>...</td>
      <td>51028.0</td>
      <td>63909.0</td>
      <td>31214.0</td>
      <td>3871.0</td>
      <td>1481.0</td>
      <td>1306.0</td>
      <td>836.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">EU28</th>
      <th>MIO_M3</th>
      <td>0.000</td>
      <td>10691.661</td>
      <td>10030.229</td>
      <td>13574.363</td>
      <td>8628.138</td>
      <td>7960.478</td>
      <td>8283.820</td>
      <td>11233.747</td>
      <td>9064.757</td>
      <td>7553.156</td>
      <td>...</td>
      <td>1348.0</td>
      <td>1690.0</td>
      <td>827.0</td>
      <td>102.0</td>
      <td>39.0</td>
      <td>33.0</td>
      <td>21.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>0.000</td>
      <td>389101.459</td>
      <td>354658.869</td>
      <td>480561.792</td>
      <td>320281.116</td>
      <td>298208.208</td>
      <td>301950.381</td>
      <td>410193.681</td>
      <td>334940.412</td>
      <td>285260.821</td>
      <td>...</td>
      <td>51028.0</td>
      <td>63909.0</td>
      <td>31214.0</td>
      <td>3871.0</td>
      <td>1481.0</td>
      <td>1306.0</td>
      <td>836.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>PL</th>
      <th>MIO_M3</th>
      <td>3652.268</td>
      <td>3949.365</td>
      <td>2968.328</td>
      <td>3572.638</td>
      <td>2664.000</td>
      <td>3189.000</td>
      <td>2361.000</td>
      <td>2279.000</td>
      <td>1888.000</td>
      <td>1714.000</td>
      <td>...</td>
      <td>470.0</td>
      <td>337.0</td>
      <td>196.0</td>
      <td>102.0</td>
      <td>37.0</td>
      <td>33.0</td>
      <td>21.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">UK</th>
      <th>EU28</th>
      <th>TJ_GCV</th>
      <td>0.000</td>
      <td>420602.492</td>
      <td>335297.099</td>
      <td>451297.756</td>
      <td>371249.294</td>
      <td>480219.234</td>
      <td>356031.688</td>
      <td>351339.000</td>
      <td>416826.098</td>
      <td>529743.614</td>
      <td>...</td>
      <td>239985.0</td>
      <td>96238.0</td>
      <td>76059.0</td>
      <td>52951.0</td>
      <td>38428.0</td>
      <td>36117.0</td>
      <td>23623.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">IE</th>
      <th>MIO_M3</th>
      <td>3437.000</td>
      <td>2852.223</td>
      <td>2040.651</td>
      <td>1652.587</td>
      <td>2004.000</td>
      <td>4234.000</td>
      <td>4245.621</td>
      <td>4371.000</td>
      <td>4521.807</td>
      <td>4631.781</td>
      <td>...</td>
      <td>2160.0</td>
      <td>1592.0</td>
      <td>990.0</td>
      <td>550.0</td>
      <td>97.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>135059.000</td>
      <td>112418.229</td>
      <td>80961.250</td>
      <td>65565.095</td>
      <td>79082.000</td>
      <td>168442.000</td>
      <td>167088.688</td>
      <td>172594.000</td>
      <td>178557.098</td>
      <td>183928.022</td>
      <td>...</td>
      <td>88099.0</td>
      <td>64933.0</td>
      <td>40261.0</td>
      <td>22489.0</td>
      <td>3946.0</td>
      <td>106.0</td>
      <td>190.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">NL</th>
      <th>MIO_M3</th>
      <td>11289.702</td>
      <td>7952.534</td>
      <td>6858.764</td>
      <td>8268.502</td>
      <td>5812.330</td>
      <td>5759.147</td>
      <td>3657.000</td>
      <td>4045.000</td>
      <td>4380.000</td>
      <td>5815.000</td>
      <td>...</td>
      <td>3335.0</td>
      <td>939.0</td>
      <td>676.0</td>
      <td>354.0</td>
      <td>385.0</td>
      <td>394.0</td>
      <td>548.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>TJ_GCV</th>
      <td>376389.404</td>
      <td>265130.958</td>
      <td>228665.558</td>
      <td>275665.075</td>
      <td>193778.294</td>
      <td>192005.234</td>
      <td>121921.000</td>
      <td>134856.000</td>
      <td>146025.000</td>
      <td>193866.000</td>
      <td>...</td>
      <td>111186.0</td>
      <td>31305.0</td>
      <td>22537.0</td>
      <td>11802.0</td>
      <td>12836.0</td>
      <td>13136.0</td>
      <td>18270.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>186 rows × 31 columns</p>
</div>



### Russian Partners


```python
russian_partners_sum_df= partners_sum_df.query('partner=="RU"')
```

### Russian Partner Bar Plot





```python
russian_partners_sum_df.plot(kind='bar', figsize = (20,10))
```




    <AxesSubplot:xlabel='partner,geo,unit'>




    
![png](output_57_1.png)
    


### Russian Partner Line Graph

Only will include partners that hvae traded for 30 years.  This is a good indicator of long term fossil fuel dependency.


```python
russian_partners_sum_df.plot(kind='line', figsize = (20,10))
```




    <AxesSubplot:xlabel='partner,geo,unit'>




    
![png](output_59_1.png)
    

