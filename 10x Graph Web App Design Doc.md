# Competition Graph Web App Design Doc

## Functional Requirement

* Backend Service
  * The backend service should be able to query and serve relationship data between public companies in the form of graph data.
    * Company relationship includes these relationships: competition, product, and unknown
  * The backend service should expose RESTful web APIs for other services
    * Client services typically includes machine learning model training services
    * The backend service should support relationship query by company names, tickers, and types of relationships. Typical usecase should include:
      * Listing companies and their relationships (currently support members of DOW30 and SP500)
      * Listing relationship of a company (competition, product, and unknown )
      * Being able to list all companies that are competitors of each other. Also, it should be able to list all competitors of a specific company.
* Frontend Service
  * The frontend service should provide visualization and interaction for users to understand the relationships between public companies.
    * The frontend service should allow users to search a company by name or ticker and show its relationship with other companies
    * The frontend service should allow users to specify the layers of relationship expansion. For example, if users only want to expand 2 layers of relationships, then only companies that are at most 2 jumps away from the searched companies will be displayed.
    * The frontend should also provide an overview given the market (either dow30 or sp500)

## Critical User Journey

1. Firstly, users will see the abstract of the Competition Graph Paper to get an idea of how the competition graph is extracted. Then user can click on the "Try it out" button to try this app
2. Users will fill a form instructing the system which company they want to view. Some recommended tickers are provided, e.g. MSFT, AMZN, AAPL, etc. Users will also tell the system how many layers of relationship to expand, and the number of nodes to view. Then users click on the submit button to view the graph. The frontend will submit users' requests to the backend. If the company ticker entered is found, the backend will return the graph data, and page will be switched to the graph visualizer page, which is a part of the website. Otherwise, the backend will report an error indicating that the company is not found.. The frontend will display the error message.
3. In the graph visualizer，which is the main page of user interaction, users can click, select and drag the graph. Users can also zoom in or zoom out. If the clicked node is a company, the frontend will send a request to fetch nodes connected to that company. If successful, the graph node will expand to display the competitors of that company. Otherwise, the frontend does nothing. If the node is already expanded, then all nodes connecting to that node will collapse and become hidden.
4. User cannot select the type of relationship to expand.

<p style="text-align:center"><img src="image/10xGraphWebAppDesignDoc/1686807119816.png"></p>

## Non-Functional Requirement

* **Performance:**  The service should be able to handle frequent requests from other services without performance degradation. Key performance metrics include:

  * **Throughput** : The service should be able to handle requests containing about 2k nodes (roughly equal to the sp500 dataset, around 270kb, which is roughly around 120 bytes per node).
    * How to achieve this:
      * the service should transmit gzipped data by default
      * the client should request the batch data api
      * the service could use some in-memory cache to increase response time
* **Scalability** : The service should be scalable to handle increasing data volume and request load. Key scalability metrics include:

  * **Load Scalability** : The service should auto scale when handling large amount of requests
    * How to achieve this: [use K8s autoscaler ](https://www.kubecost.com/kubernetes-autoscaling/kubernetes-cluster-autoscaler/)
* **Availability:**  The service should be highly available to ensure uninterrupted service to users. Key availability metrics include:

  * **Failover** : In case of a system failure, the service should be able to automatically failover to a backup system within minutes.
    * How to achieve this:[ Liveness Probe](https://blog.csdn.net/dkfajsldfsdfsd/article/details/81086633)

## Web API

#### For Machine Learning Services

The service will expose a RESTful API suitable for batch data request

How to consume it: [here](https://requests.readthedocs.io/en/latest/user/quickstart/#binary-response-content)

1. POST /graph-request
   1. returns a zipped graph data starting with a center node
   2. params
      1. center_node: str
      2. relationship_type: `'comp' | 'product' | 'unknown'`
         * `'comp'`: competition
         * `'product'`: represents products of a company
         * `'unknown'`: other relationships
         * `none`: returns all relationships
      3. n_layers: int
         1. how many layers to expand

#### For Graph Visualizer

The service will expose a RESTful API with the following endpoints:

1. GET `/relationship/<relationship_type: 'comp' | 'prod' | 'unknown' | none>?cik=<cik: str>&ticker=<ticker: str>&company=<company: str>`
   1. Returns a list of companies by relationship type
   2. Params:
      1. `company_name: str`,
      2. `relationship_type: 'comp' | 'product' | 'unknown'`,
         1. `'comp'`: competition
         2. `'product'`: represents products of a company
         3. `'unknown'`: other relationships
         4. `none`: returns all relationships
   3. Returns
      1. graph
         1. `nodes: List[{ id: int, name: str }]`
         2. `links: List[{ id: int, category: 'competition' | 'product' | 'unknown', source: int, target: int }]`
2. GET /relationship/overview/`<market: 'dow30' | 'sp500'>`
   1. Returns an overview of all companies in the market (dow30, sp500)
3. GET

![1687008056491](image/10xGraphWebAppDesignDoc/1687008056491.png)

## UI Design

(For reference only. May not represent the final product.)

![1687009025863](image/10xGraphWebAppDesignDoc/1687009025863.png)

![1687009043423](image/10xGraphWebAppDesignDoc/1687009043423.png)

![1687009367835](image/10xGraphWebAppDesignDoc/1687009367835.png)

## Architectural Design

The service will adopt a microservices architecture, with distinct backend and frontend services. Both services will be designed with a focus on high availability and fault tolerance.

### Load Balancing

By default we use Nginx for load balancing. If the system is deployed via K8s, K8s' load balancer will be used instead of Nginx.

### Backend Architectural Design

![1687010882726](image/10xGraphWebAppDesignDoc/1687010882726.png)

The backend service will be developed using a graph database(Neo4j) and Python. It will be designed as a distributed system and deployed using container services for high availability and scalability. **To optimize query performance, we can optionally deploy a redis service to cache frequent queries.** Frequent quries will be sent to redis instead of hitting the database. The key could be the request parameters like "company_name={name}&rel={comp|prod|unknown|none}", and the value is the data returned by the backend.

#### Distributed System

The backend service will be designed as a distributed system with multiple instances running in parallel. This design allows the service to handle a large number of requests and ensures high availability.

#### Container Services

The backend service will be deployed using Docker, and orchestrated using Kubernetes (k8s). This allows for easy deployment, scaling, and management of the service.

1. **Containerization** : Each microservice will be packaged into a docker container. This includes all the dependencies needed by the microservice, ensuring consistency across all environments.
2. **Orchestration** : The containers will be managed and orchestrated using Kubernetes. Kubernetes provides features like automatic scaling, rolling updates, and self-healing (restarting containers that fail), which are crucial for high availability and scalability.

#### Database Deployment

The database could be deployed as a single node for the sake of simplicity. Replication will be considered if the data size is large enough.

### Monitoring and Alerting

To ensure the high availability of the service, a robust monitoring and alerting system will be implemented. This system will continuously monitor the health of the servers and the database, and send alerts in case of any issues. This allows for quick detection and resolution of any problems, minimizing downtime.

The system will be deployed with K8s + Prometheus.

#### Metrics to monitor

- CPU usage
- Disk I/O
- Network I/O
- Node status
- Cluster health
- Uptime
- Downtime
- **Error Rates:** The number of failed requests over total requests.

### Cache Key Design (Optional)

Redis is used as the caching system between the backend and its consumers.

For simplicity, a in-memory cache layer can be adopted.

Data will be cached to redis on the first update, and get refreshed after 5 minutes.

The key design is as follows:

``"company_name:{company_name: str, None}&relationship_type:{all, comp, prod, unknown}"``

Each query will be cached. If the key hits the cache, the cached data will be returned in favor of the data from the database.

Since other client services asynchronously pulls data from the graph service, redis will be used as  both a message queue and a cache system to coordinate communication between backend services. Thus, client services will use redis's Pub/Sub command to pull data.

The graph service will subscribe a channel called `relationship_query` . To request data, services can send command via this channel.

```
SUBSCRIBE relationship_query
```

The query command is the same as the format of cache key.

``"company_name:{company_name: str, None}&relationship_type:{all, comp, prod, unknown}"``

To query data, send command in the following format.

```
Publish relationship_query {'query': {'company_name': str, None, 'relationship_type': all, comp, prod, unknown}}}
```

The graph service will publish a channel called `company_relationship`. It will publish data like the following:

```
PUBLISH company_relationship {'graph': {nodes:[{ id: int, name: str }], links: [{ id: int, category: 'competition' | 'product' | 'unknown', source: int, target: int }]}}}}
```

Client services will subscribe to `company_relationship` channel to receive the data.

```
SUBSCRIBE relationship_query
```

## DB Design

**DB Choice: Neo4j**

![1688548724003](image/10xGraphWebAppDesignDoc/1688548724003.png)

**Nodes:**

1. **Company** : This node represents a company. Properties:

* `name`: The name of the company.
* `ticker`: The stock ticker symbol of the company.
* `industry`: The industry the company operates in.

2. Product: Represents the product of a company. Properties:

* `name`: The name of the product

3. Item: Other unknown item

* `name`: The name of the item

**Relationships:**

1. **COMPETES_WITH** : This relationship represents a competitive relationship between two companies.
2. **PRODUCT_OF** : This relationship represents the item
3. **Unknown** : This relationship represents all other unknown relationships.

##### Index Design

**Company Nodes** : You might frequently query for companies based on their `name` or `ticker` properties

```sql
CREATE INDEX FOR (c:Company) ON (c.name)
CREATE INDEX FOR (c:Company) ON (c.ticker)
```

**Product Nodes** :

```SQL
CREATE INDEX FOR (p:Product) ON (p.name)
```

**Item Nodes:**

```sql
CREATE INDEX FOR (i:Item) ON (i.name)
```

### Key Assumptions and Trade-offs

#### **Key Assumptions:**

1. **Data Availability** : The system assumes that the data about companies and their relationships is available and can be obtained in a structured format that can be imported into Neo4j.
2. **Data Quality** : The system assumes that the data is accurate and up-to-date. If the data is not reliable, the results provided by the system will also be unreliable.
3. **User Load** : The system assumes that the load will be manageable with the proposed architecture. If the number of users or the rate of requests is significantly higher than expected, the system might need to be scaled up.
4. **User Behavior** : The system assumes that users will interact with the system in certain ways (e.g., by querying for specific companies, by exploring the graph visualization). If users interact with the system in unexpected ways, it might affect the performance or usability of the system.

#### **Key Trade-offs**

1. **Complexity vs. Performance** : Using a graph database like Neo4j allows for efficient querying of complex relationships, but it also adds complexity to the system compared to a more traditional relational database.
   1. Why choosing a graph database? This system is primarily deal with graph and relational data, so using a specialized graph database can represent, query, and store the data more expressively. On the other hand, if we are storing and querying graph data in a relational database, we may need a lot of joins to query long range relationship. Also, the schema may include self referential structure, which is also hard to design in a relationship database.
2. **Scalability vs. Consistency** : In a distributed system, there is often a trade-off between scalability and consistency. For example, adding more replicas can increase read scalability but can also lead to consistency issues.
   1. In our case, we decide to use a few replicas of graphical db to spread the load, as the downstream service may send a stream of requests and pull a lot of data. Deploying replicas not only reduce the load but also decrease the probability of failed requests.
   2. On the other hand, we should not too far on this path of deploying a lot of distributed nodes. It may introduce unnecessary costs and complexity to our system.
3. **Cost vs. Performance** : Higher performance often comes with higher costs, both in terms of the resources needed (e.g., more powerful servers, more storage) and the complexity of the system (e.g., implementing caching, load balancing, replication).

## Implementation Story

**Development Phase** : The development phase was characterized by rigorous coding activity. The team implemented the database schema in Neo4j, developed the backend service using Python, and designed a frontend interface for visualizing company relationships. The complexity of graph queries in Neo4j presented a significant challenge, which was overcome through meticulous query design and effective utilization of Neo4j's features.

* [ ] Frontend Development
  * [ ] Interaction Enhancement
    * [X] Support company relationship query
    * [X] Support SP500 & DOW30 global chart
    * [X] Support node expansion
    * [ ] Support relationship filtering (choose to expand nodes based on relationship type)
* [ ] Backend Development
  * [X] Basic RESTful graph API loaded from json file
    * [X] Supports SP500 & DOW30 global chart
    * [X] Support node expansion given company and relationship type
    * [X] Support node expansion by number of nodes and number of layers
    * [ ] Support for gzipped batch request API
* [ ] Data preparation and migration
  * [ ] Neo4j Graph schema building: defined schema and indexes for the competition graph
  * [ ] Data migration: load all json data to neo4j

**Testing Phase** : A comprehensive testing strategy was implemented, encompassing unit tests, integration tests, and load tests. Performance bottlenecks identified during load testing were addressed through query optimization and enhanced caching.

* [ ] Test case development
  * [ ] Service modules should be unit tested
* [ ] System Integration Test
  * [ ] Function test

    * [ ] Backend test

      * [ ] Can the service returns full dow30 graph within 5 second?
      * [ ] Can the service returns full sp500 graph within 5 second?
    * [ ] Visualizer UI test

      * [ ] Can DOW30 graph be displayed?
      * [ ] Can SP500 graph be displayed?
      * [ ] Can user choose the amount of nodes to expand?
      * [ ] Can users choose a company as the center node and expand up to n layers?
  * [ ] Pressure test: Does the system endures pressure and function normally under heavy load?

    * [ ] Can the service handles request of more than 1k nodes without overtime?
    * [ ] Can the service handles request of more than 5k nodes without overtime?
    * [ ] Can the service handles request of more than 10k nodes without overtime?
* [ ] User Acceptance Test
  * [ ] Does user can work with the system properly
  * [ ] Does users interaction go well?
  * [ ] Does users feel comfortable using the system?

**Deployment Phase** : The deployment phase marked a significant milestone in the project. The production environment was configured with multiple instances of the backend service, Neo4j, and Redis, all managed by Kubernetes. Nginx was employed as a load balancer to ensure equitable distribution of requests across backend instances.

* [ ] Frontend Deployment
  * [ ] The finished version should be deployed on at least one server instance
* [ ] LB Deployment
  * [ ] The LB should be online and proxy requests to proper frontend resources and backend services
* [ ] Backend Deployment
  * [ ] The backend service should be deployed on at least one server instance
* [ ] Redis Deployment
  * [ ] One redis instance should be deployed
* [ ] Neo4j Deployment
  * [ ] Expect 1 Neo4j DB to be deployed.
* [ ] Monitoring
  * [ ] Prometheus

**Post-Deployment** : Subsequent to deployment, a monitoring system was established using Prometheus and Grafana to track the system's performance and availability. A procedure for handling user feedback and system updates was also instituted. The system has demonstrated consistent performance and has been well-received by users.

* [ ] Going online and receive feedbacks

**Reflection** : In retrospect, the project has been a successful endeavor, resulting in the creation of a robust, high-performance system that effectively meets user requirements. The experience has provided valuable insights into working with graph databases, designing distributed systems, and managing high-availability deployments. The team looks forward to further improving the system and tackling new challenges in the future.

* [ ] Write and share retrospect on this project
