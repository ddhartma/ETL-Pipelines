# ETL Pipelines
Data comes in all shapes, formats and sizes. It's a data engineers job to make sure that all of this information is accessible and ready for analysis. One of the major responibilities of a data engineer is called ***pipelining***.
## Introduction
Data pipeline is a generic term for moving data from one place to another. For example, it could be moving data from one server to another server, database to database text file to csv file or text file to a database.

#### ETL
An ETL pipeline is a specific kind of data pipeline and very common. ETL stands for Extract, Transform, Load. Imagine that you have a database containing web log data. Each entry contains the IP address of a user, a timestamp, and the link that the user clicked.

#### ELT
ELT (Extract, Load, Transform) pipelines have gained traction since the advent of cloud computing. Cloud computing has lowered the cost of storing data and running queries on large, raw data sets. Many of these cloud services, like Amazon Redshift, Google BigQuery, or IBM Db2 can be queried using SQL or a SQL-like language. With these tools, the data gets extracted, then loaded directly, and finally transformed at the end of the pipeline.

However, ETL pipelines are still used even with these cloud tools. Oftentimes, it still makes sense to run ETL pipelines and store data in a more readable or intuitive format. This can help data analysts and scientists work more efficiently as well as help an organization become more data driven.

##### The route
1. Extract data from different sources such as:
  - csv files
  - json files
  - APIs
2. Transform data
  - combining data from different sources
  - data cleaning
  - data types
  - parsing dates
  - file encodings
  - missing data
  - duplicate data
  - dummy variables
  - remove outliers
  - scaling features
  - engineering features
3. Load
  - send the transformed data to a database
4. ETL Pipeline
  - code an ETL pipeline


## The Data
The data comes from two sources:
- [World Bank Indicator Data](https://data.worldbank.org/indicator) - This data contains socio-economic indicators for countries around the world. A few example indicators include population, arable land, and central government debt.
- [World Bank Project Data](https://datacatalog.worldbank.org/dataset/world-bank-projects-operations) - This data set contains information about World Bank project lending since 1947.

## Extract Data
#### Typical formats
- ***CSV***: CSV stands for comma-separated values. These types of files separate values with a comma, and each entry is on a separate line. Oftentimes, the first entry will contain variable names.
- ***JSON***: JSON is a file format with key/value pairs. It looks like a Python dictionary.
- ***XML***: XML stands for Extensible Markup Language. XML is very similar to HTML at least in terms of formatting. The main difference between the two is that HTML has pre-defined tags that are standardized. In XML, tags can be tailored to the data set.
- ***SQL databases***: SQL databases store data in tables using primary and foreign keys.
- ***Text Files***: Whereas CSV, JSON, XML, and SQL data are organized with a clear structure, text is more ambiguous.

#### Extracting Data from the Web
- Data from the web can often be extracted by using an API (Application Programming Interface).
- APIs generally provide data in either JSON or XML format.

### [Extracting CSV data](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_json.html)
Open the Jupyter Notebook ***./extract/extract_CSV.ipynb*** for csv handling.
Read csv tricks:
- read_csv simple:
  ```
  import pandas as pd
  df_projects = pd.read_csv('projects_data.csv')
  ```
- In case of DType warning (Pandas could not automatically figure out the data type for each column (ie integer, string, etc.) use ```dtype=str```

  ```
  df_projects = pd.read_csv('projects_data.csv', dtype=str)
  ```

- In case of ```ParserError: Error tokenizing data```:
  ```
  f = open('population_data.csv')
  for i in range(10):
      line = f.readline()
      print('line: ', i,  line)
  f.close()
  ```
  - Read data line by line
  - use skiprow to overcome header problems in csv

  ```
  df_population = pd.read_csv('population_data.csv', skiprows=4)
  ```


### [Extracting JSON data](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_json.html)
Open the Jupyter Notebook ***./extract/extract_JSON_XML.ipynb*** for JSON handling.
Read JSON tricks:
- read JSON lines:
  ```
  def print_lines(n, file_name):
    f = open(file_name)
    for i in range(n):
        print(f.readline())
    f.close()

  print_lines(1, 'population_data.json')
  ```
- read JSON via pandas.read_json()
  ```
  df_json = pd.read_json('population_data.json', orient='records')
  ```
- read JSON via JSON library
  ```
  import json
  with open('population_data.json') as f:
      json_data = json.load(f)
  ```

### [Extracting XML data](https://www.crummy.com/software/BeautifulSoup/)
Open the Jupyter Notebook ***./extract/extract_JSON_XML.ipynb*** for XML handling.
- Use BeautifulSoup
  ```
  # import the BeautifulSoup library
  from bs4 import BeautifulSoup

  # open the population_data.xml file and load into Beautiful Soup
  with open("population_data.xml") as fp:
      soup = BeautifulSoup(fp, "lxml") # lxml is the Parser type
  ```
  ```
  # output the first 5 records in the xml file
  # this is an example of how to navigate the XML document with BeautifulSoup

  i = 0
  # use the find_all method to get all record tags in the document
  for record in soup.find_all('record'):
      # use the find_all method to get all fields in each record
      i += 1
      for record in record.find_all('field'):
          print(record['name'], ': ' , record.text)
      print()
      if i == 5:
          break
  ```

- Convert XML to DataFrame (via dictionary)
  ```
  data_dictionary = {'Country or Area':[], 'Year':[], 'Item':[], 'Value':[]}

  for record in soup.find_all('record'):
      for record in record.find_all('field'):
          data_dictionary[record['name']].append(record.text)

  df = pd.DataFrame.from_dict(data_dictionary)
  df = df.pivot(index='Country or Area', columns='Year', values='Value')
  df.reset_index(level=0, inplace=True)
  ```


### Extracting SQL databases [SQLite](https://www.sqlite.org/about.html) or [SQL Alchemy](https://www.sqlalchemy.org/)
Open the Jupyter Notebook ***./extract/extract_SQL.ipynb*** for SQL handling.
- Connect to SQLite3 Database via
  ```
  import sqlite3
  import pandas as pd

  # connect to the database
  conn = sqlite3.connect('population_data.db')

  # run a query
  pd.read_sql('SELECT * FROM population_data', conn)
  ```

- Connect to SQLAlchemy via
  ```
  import pandas as pd
  from sqlalchemy import create_engine

  ###
  # create a database engine
  # to find the correct file path, use the python os library:
  # import os
  # print(os.getcwd())
  #
  ###

  engine = create_engine('sqlite:////home/workspace/3_sql_exercise/population_data.db')
  pd.read_sql("SELECT * FROM population_data", engine)
  ```
  Check the note book for further queries.

### Extracting data via APIs
Open the Jupyter Notebook ***./extract/extract_API.ipynb*** for API handling.
- An API allows one to execute code on the (World Bank) server without getting direct access.
In general, you access APIs via the web using a web address. Within the web address, you specify the data that you want. To know how to format the web address, you need to read an API's documentation. Some APIs also require that you send login credentials as part of your request. The World Bank APIs are public and do not require login credentials.

  ```
  http://api.worldbank.org/v2/countries/ + list of country abbreviations separated by ; + /indicators/ + indicator name + ? + options
  ```
  where options can include
    - per_page - number of records to return per page
    - page - which page to return - eg if there are 5000 records and 100 records per page
    - date - filter by dates
    - format - json or xml

    and a few other options that you can read about [here](https://datahelpdesk.worldbank.org/knowledgebase/articles/898581-api-basic-call-structure).

- The Python requests library makes working with APIs relatively simple.
  ```
  import requests
  import pandas as pd

  url = 'http://api.worldbank.org/v2/countries/br;cn;us;de/indicators/SP.POP.TOTL/?format=json&per_page=1000'
  r = requests.get(url)
  r.json()
  ```


## Transform Data
- Data scientists claim that they spend 80% of their time transforming data.
- Data Transformation means:
  - Combining & cleaning data
  - Working with encodings
  - Removing dublicate rows
  - Dummy variables
  - Removing outliers
  - Normalizing data
  - Engineer new features

### Combinig data from different datasets
e.g. you have top combine data from a JSON file with data from a CSV file
Open the Jupyter Notebook ***./transform/transform_comnbine.ipynb*** for DataFrame Combinations.

- Use the [concat](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html) method to add df_electricity at the end of df_rural
- Only possible if both DataFrames have the same column structure
  ```
  df = pd.concat([df_rural, df_electricity])
  ```

- Use the [melt](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.melt.html) method to unpivot a DataFrame from wide to long format, optionally leaving identifiers set.
  ```
  df_rural_melt = pd.melt(df_rural, id_vars=['Country Name', 'Country Code', 'Indicator Name', 'Indicator Code'],\
                  var_name = 'Year', value_name='Rural_Value')
  ```
- Use the [merge](https://pandas.pydata.org/pandas-docs/version/0.23/generated/pandas.DataFrame.merge.html) method to join DataFrames
  ```
  df_merge = df_rural_melt.merge(df_electricity_melt, how='outer', on=['Country Name', 'Country Code', 'Year'])
  ```

### Cleaning data
Open the Jupyter Notebook ***./transform/transform_cleaning.ipynb*** for Data Cleaning.

- Dirty data tends to come from a number of sources including:
  - missing values
  - data entry mistakes
  - dublicate data
  - incomplete records
  - inconsistencies between different datasets
  - incorrect encodings


- Drop duplicates via pandas [drop_dublicates](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop_duplicates.html) method
  ```
  df_indicator[['Country Name', 'Country Code']].drop_duplicates()
  ```
- Get unique column enntries of DataFrame via pandas [unique](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.unique.html) method
  ```
  df_projects['countryname'].unique()
  ```
- Apply a lambda function to all rows of one column for cleaning, e.g.

  split column values:
  ```
  df_projects['Official Country Name'] = df_projects['countryname'].apply(lambda x : x.split(';')[0])
  ```

  replace colume values:
  ```
  df_projects['totalamt'] = df_projects['totalamt'].apply(lambda x : x.replace(',', ''))
  ```
  or simply
  ```
  df_projects['totalamt'] = df_projects['totalamt'].replace(',', '')
  ```


### Datatypes
Open the Jupyter Notebook ***./transform/transform_datatypes.ipynb*** for handling datatypes.
- When reading in a data set, pandas will try to guess the data type of each column like float, integer, datettime, bool, etc. In Pandas, strings are called "object" dtypes. However, Pandas does not always get this right.
- With messy data, you might find it easier to read in everything as a string; however, you'll sometimes have to convert those strings to more appropriate data types. When you output the dtypes of a dataframe, you'll generally see these values in the results:
  - float64
  - int64
  - bool
  - datetime64
  - timedelta
  - object


- To get the datatypes of each column use
  ```
  df_projects.dtypes
  ```

- Conversion to string during reading
  ```
  df_projects = pd.read_csv('../data/projects_data.csv', dtype=str)
  ```

- Conversion to numeric values
  ```
  df_projects['totalamt'] = pd.to_numeric(df_projects['totalamt'])
  ```
- Important datatype converion methods
  - [astype](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.DataFrame.astype.html#pandas.DataFrame.astype)
  - [to_datetime](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.to_datetime.html#pandas.to_datetime)
  - [to_numeric](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.to_numeric.html#pandas.to_numeric)
  - [to_timedelta](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.to_timedelta.html#pandas.to_timedelta)

### Parsing Dates
Open the Jupyter Notebook ***./transform/transform_parsing_dates.ipynb*** for parsing dates.
- Another common data transformation involves parsing dates. Parsing generally means that you start with a string and then transform that string into a different data type. In this case, that means taking a date in the format of a string and transforming the string into a date type.
  ```
  import pandas as pd
  parsed_date = pd.to_datetime('January 1st, 2017')
  ```

  In order to parse it use:
  ```
  parsed_date.month
  parsed_date.year
  parsed_date.second
  ```

- Sometimes date string are formatted in unexpected ways. For example, in the United States, dates are given with the month first and then the day. That is what pandas expects by default. However, some countries write the date with the day first and then the month.
  ```
  parsed_date = pd.to_datetime('5/3/2017 5:30')
  parsed_date.month

  parsed_date = pd.to_datetime('3/5/2017 5:30', format='%d/%m/%Y %H:%M')
  parsed_date.month

  parsed_date = pd.to_datetime('5/3/2017 5:30', format='%m/%d/%Y %H:%M')
  parsed_date.month
  ```

- Order to parse a whole DataFrame column use e.g.
  ```
  df_projects['boardapprovaldate'] = pd.to_datetime(df_projects['boardapprovaldate'], format="%Y-%m-%dT%H:%M:%SZ")
  df_projects['boardapprovaldate'].dt.year
  df_projects['boardapprovaldate'].dt.month
  df_projects['boardapprovaldate'].dt.day
  df_projects['boardapprovaldate'].dt.weekday
  df_projects['boardapprovaldate'].dt.hour
  df_projects['boardapprovaldate'].dt.minute
  df_projects['boardapprovaldate'].dt.second
  ```

- Check this link for [Python's strftime directives](https://strftime.org/)


### Matching Encodings
Open the Jupyter Notebook ***./transform/transform_encodings.ipynb*** for handling encodings.
- Encodings are a set of rules mapping string characters to their binary representations. Python supports dozens of different encoding as seen here in this [link](https://docs.python.org/3/library/codecs.html#standard-encodings). Because the web was originally in English, the first encoding rules mapped binary code to the English alphabet.
The English alphabet has only 26 letters. But other languages have many more characters including accents, tildes and umlauts. As time went on, more encodings were invented to deal with languages other than English. The utf-8 standard tries to provide a single encoding schema that can encompass all text.
The problem is that it's difficult to know what encoding rules were used to make a file unless somebody tells you. The most common encoding by far is utf-8. Pandas will assume that files are utf-8 when you read them in or write them out.

- Use chardet for autodetection of encodings
  ```
  !pip install chardet

  # import the chardet library
  import chardet

  # use the detect method to find the encoding
  # 'rb' means read in the file as binary
  with open('../data/population_data.csv', 'rb') as file:
      print(chardet.detect(file.read()))
      encoding = chardet.detect(file.read())

  df = pd.read_csv('../data/population_data.csv', skiprows=4, encoding=encoding['encoding'])
  ```

### Missing values
Open the Jupyter Notebook ***./transform/transform_imputations.ipynb*** for handling missing values.
In most cases a machine learning algorithm won't work with missing values.

Common methods to handle missing values
  1. Deleting entire rows
  2. Imputing values by mean substitution
    Here are some links
    - [groupby](https://pandas.pydata.org/pandas-docs/version/0.23/generated/pandas.DataFrame.groupby.html)
    - [transform](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.transform.html)
    - [fillna](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.DataFrame.fillna.html)
    ```
    df_melt['GDP_filled'] = df_melt.groupby('Country Name')['GDP'].transform(lambda x: x.fillna(x.mean()))
    ```

  3. Imputing values by mode substitution

  4. Imputing values by median substitution

  5. Imputing values by [Forward Fill](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.DataFrame.fillna.html) (time series data), use neighboring cells to fill in the data.

  The pandas fillna method has a forward fill option. For example, if you wanted to use forward fill on the GDP dataset, you could execute ```df_melt['GDP'].fillna(method='ffill')```. However, there are two issues with that code.

    - You want to first make sure the data is sorted by year
    - You need to group the data by country name so that the forward fill stays within each country

  ```
  df_melt['GDP_ffill'] = df_melt.sort_values('year').groupby('Country Name')['GDP'].fillna(method='ffill')
  ```
  6. Imputing values by [Backward Fill](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.DataFrame.fillna.html) (time series data), use neighboring cells to fill in the data
  ```
  df_melt['GDP_bfill'] = df_melt.sort_values('year').groupby('Country Name')['GDP'].fillna(method='bfill')
  ```

### Data Dublicates
Open the Jupyter Notebook ***./transform/transform_dublicates.ipynb*** for handling dublicate values.
- A data set might have duplicate data: in other words, the same record is represented multiple times. Sometimes, it's easy to find and eliminate duplicate data like when two records are exactly the same. At other times, duplicate data is hard to spot.
- Drop duplicates via pandas [drop_dublicates](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop_duplicates.html) method
  ```
  df_indicator[['Country Name', 'Country Code']].drop_duplicates()
  ```
- Sometimes you have to look for certain strings in order to find dublicates, so use ```str.contains```
  ```
  projects[projects['countryname'].str.contains('Yugoslavia')]
  ```

### Dummy Variables
Open the Jupyter Notebook ***./transform/transform_dummy.ipynb*** for creating dummy variables.
- Dummy variables are often needed for categorical columns in order to make predictions via Linear Regression. The reasoning behind these transformations is that machine learning algorithms read in numbers not text. Text needs to be converted into numbers. If you have five categories, you only really need four features. For example, if the categories are "agriculture", "banking", "retail", "roads", and "government", then you only need four of those five categories for dummy variables. This topic is somewhat outside the scope of a data engineer.
In some cases, you don't necessarily need to remove one of the features. It will depend on your application. In regression models, which use linear combinations of features, removing a dummy variable is important. For a decision tree, removing one of the variables is not needed.
- Often dummy variables are one hot vectors of ```length = number of categrocal vars - 1```
- Pandas makes it relatively easy to create dummy variables; however, oftentimes you'll need to clean the data first. Use ```replace``` strings in columns or ```split``` to remove certain substrings from column-row. Create dummies for categorical subset. Add numerical columns via ```concat``` method for example.

- Easy method in pandas [get_dummies](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.get_dummies.html)
  ```
  dummies = pd.DataFrame(pd.get_dummies(sector['sector1']))
  ```
- To reduce dimension of categorical subset it might be useful to combine a few of them together. For example, in the dataset there are various categories with the term "Energy". And then there are other categories that seem related to energy but don't have the word energy in them like "Thermal" and "Hydro". Some categories have the term "Renewable Energy", depending on your questions you might want to combine these sets to one category.
  ```
  sector.loc[sector['sector1_aggregates'].str.contains('Energy', re.IGNORECASE).replace(np.nan, False),'sector1_aggregates'] = 'Energy'
  sector.loc[sector['sector1_aggregates'].str.contains('Transportation', re.IGNORECASE).replace(np.nan, False),'sector1_aggregates'] = 'Transportation'
  ```


## Setup Instructions

The following is a brief set of instructions on setting up a cloned repository.

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites: Installation of Python via Anaconda and Command Line Interaface
- Install [Anaconda](https://www.anaconda.com/distribution/). Install Python 3.7 - 64 Bit
- If you need a good Command Line Interface (CLI) under Windowsa you could use [git](https://git-scm.com/). Under Mac OS use the pre-installed Terminal.

- Upgrade Anaconda via
```
$ conda upgrade conda
$ conda upgrade --all
```

- Optional: In case of trouble add Anaconda to your system path. Write in your CLI
```
$ export PATH="/path/to/anaconda/bin:$PATH"
```

### Clone the project
- Open your Command Line Interface
- Change Directory to your project older, e.g. `cd my_github_projects`
- Clone the Github Project inside this folder with Git Bash (Terminal) via:
```
$ git clone https://github.com/ddhartma/Data-Science-Web-Development.git
```

- Change Directory
```
$ cd 5_ETL_DATA_Pipelines
```

- Create a new Python environment, e.g. ds_etl. Inside Git Bash (Terminal) write:
```
$ conda create --name ds_dashboard
```

- Install the following packages (via pip or conda)
```
numpy = 1.17.4
pandas = 0.24.2
plotly = 4.6.0
Flask = 1.1.2
```

- Check the environment installation via
```
$ conda env list
```

- Activate the installed spotify_analysis environment via
```
$ conda activate ds_dashboard
```
