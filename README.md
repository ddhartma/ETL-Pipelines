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

## Extract
#### Typical formats
- ***CSV***: CSV stands for comma-separated values. These types of files separate values with a comma, and each entry is on a separate line. Oftentimes, the first entry will contain variable names.
- ***JSON***: JSON is a file format with key/value pairs. It looks like a Python dictionary.
- ***XML***: XML stands for Extensible Markup Language. XML is very similar to HTML at least in terms of formatting. The main difference between the two is that HTML has pre-defined tags that are standardized. In XML, tags can be tailored to the data set.
- ***SQL databases***: SQL databases store data in tables using primary and foreign keys.
- ***Text Files***: Whereas CSV, JSON, XML, and SQL data are organized with a clear structure, text is more ambiguous.

#### Extracting Data from the Web
- Data from the web can often be extracted by using an API (Application Programming Interface).
- APIs generally provide data in either JSON or XML format.


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
