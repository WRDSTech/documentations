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

### public

1. get_item(specific_item_key: str) -> str

* GET /api/tenk/item?id=<itemkey>

2. get_all_items_from_filing(filing_name: str) -> dict(item_number, item_content)

* GET /api/tenk/items
* Request body: {filing_name: ???}


3. get_competition_from_filing(filing_name: str) -> str

* GET /api/tenk/competition
* Request body: {filing_name: ???}

### private

4. retrieve_competition_from_item1(item1_content: str) -> str 

5. retrieve_items_from_filing(raw_filing_content: str) -> dict(item_number, item_content)


## High Level Design

**Entry**

Client -> front end server -> backend API gateway LB -> web servers 

**File is in DB**
backend API gateway LB -> web servers -> DB

**File is not in DB**
backend API gateway LB -> web servers -> computation node -> DB


## OO Design



## DB Design

### Original Filing 
Storage Choice: File systems

Schema:
- file_path: file


### Item Table
Storage: No-SQL

Schema:
- key/sharding key: {filing_name}#{part_number}#{item_number}
- value: html
- value: text
