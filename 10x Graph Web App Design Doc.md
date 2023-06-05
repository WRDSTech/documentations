# Competition Graph Web App Design Doc

## Functional Requirement

1. Extract and store competition relatioship from financial reports
2. Query and display extracted relationship
   1. support partial visualization of competition relationships:
      1. Users can choose or search a company and view its competitors
      2. Users can choose how many nodes or companies to visualize
3. Categorize different types of competition relationships
   1. Each type of competition relationship should contain different label

## Non-Functional Requirement

1. High availability
   1. The web app should provide service for 99% of the time
2. Reasonable response time
   1. The web app should present data and visualization within 500ms by average
3. Storage independence
   1. The web app itself can use different types of storage solutions: Graph DB, Relational DB, other NoSQL databases

## Scale

Max 100 queries per second (QPS) for retrieving company relationship

Storage: 750,000 filings, estimated 750k*20=1.5MM nodes in graph,
similar scale on number of edges

## Web API

The design of web API follows the RESTful standard.

GET /competitors/<company_name>

GET /competition-graph?center-node-id={node_id: int}&max-layers={max_expand_layers: int}


## Architecture



![1685946694487](image/10xGraphWebAppDesignDoc/1685946694487.png)

## OO Design

## DB Design

### Graph Files

DB Choice: 3 Files in Hard Drive

Schema:

- train2id: (e1, e2, rel)
- relation2id: (relation name, id)
- entity2id: (entity, id)
