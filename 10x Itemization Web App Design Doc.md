# Itemization Web App Design Doc

## Functional Requirement
1. Parse a SEC 10-K filing to get items
2. Get competition section from a SEC 10-K filing


## Non-Functional Requirement
1. High availability
2. Accuracy and quality of the algorithm

## Scale
Max 100 queries per second (QPS) for parsing
Max 100 QPS for retrieve existing items

Storage: 750,000 filings, estimated 750k*20=1.5MM items on file

## API

get_file_list() - internal test -> list

get_item(specific item key) -> String

get_filing_items(filing name) -> dict(item_number, item_content)

get_filing_competition(filing name) -> String


## High Level Design

Client -> front end server -> backend API gateway LB -> web servers -> DB


## OO Design



## DB Design

Table Name: Original Filing 

Storage: File systems

Schema:
- file_path: file


Item Table

Storage: No-SQL

Schema:
- key/sharding key: {filing_name}#{part_number}#{item_number}
- value: html
- value: text
