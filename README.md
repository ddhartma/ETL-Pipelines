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
- Dirty data tends to come from a number of sources including:
  - missing values
  - data entry mistakes
  - dublicate data
  - incomplete records
  - inconsistencies between different datasets
  - incorrect encodings

Open the Jupyter Notebook ***./transform/transform_cleaning.ipynb*** for Data Cleaning.
- Drop duplicates via pandas [drop_dublicates](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop_duplicates.html) method
  ```
  df_indicator[['Country Name', 'Country Code']].drop_duplicates()
  ```
- Get unique column enntries of DataFrame via pandas [unique](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.unique.html) method
  ```
  df_projects['countryname'].unique()
  ```
- Apply a lambda function to all rows of one column for cleaning
  ```
  df_projects['Official Country Name'] = df_projects['countryname'].apply(lambda x : x.split(';')[0])
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
$ cd 3_Flask
```

- Create a new Python environment, e.g. ds_dashboard. Inside Git Bash (Terminal) write:
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
