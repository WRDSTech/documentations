# Data Synchronization Tool

## Primary Goal

Achieve repeated and scheduled data synchronization from various data sources. This tool keeps your local data storage in sync with remote data sources, and helps you manage your synchronization plan, data sources, and local data storage. To support complex and customized requirements of pulling data from web apis, this tool provides template management features to help you generate customized request arguments and migration scripts.

## Some Background

Data is the foundation of a quantitative trading system, and users have to pull data from various sources on regular basis. Simple solutions exist, but they come with costs of maintainance. This tool aims to reduce the amount of maintainance workload by providing functionalities of managing data source and synchronization plans. 

This project originates from the need of pulling data from a famous public quant data provider called [Tushare](https://www.tushare.pro). This data provider serves diverse types of data. Click the link if you are interested.

## Features

1. Data synchronization planning and scheduling
2. Automatic data sychronization
3. Data source management
4. Local storage management
5. Custom web request argument customization
   1. Pulling data by calling web apis can be complicated because the request arguments are unique by each provider and may require dynamic generation. They also may depend on some other data to be generated.
