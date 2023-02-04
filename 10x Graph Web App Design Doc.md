# Competition Graph Web App Design Doc

## Functional Requirement
1. Store graph details
2. Query company relation graph 

## Non-Functional Requirement
1. High availability
2. It may fit into different storage solution

## Scale
Max 100 queries per second (QPS) for retrieving company relationship

Storage: 750,000 filings, estimated 750k*20=1.5MM nodes in graph, 
similar scale on number of edges

## API

get_surrouding_by_node(node_id, expand_number_of_layers) -> 


## High Level Design

Client -> front end server -> backend API gateway LB -> web servers -> DB


## OO Design



## DB Design


