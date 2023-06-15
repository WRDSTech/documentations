# Competition Graph Web App Design Doc

## Features

1. Extract and store competition relatioship from financial reports
2. Query and display extracted relationship
   1. support partial visualization of competition relationships:
      1. Users can choose or search a company and view its competitors
      2. Users can choose how many nodes or companies to visualize
3. Categorize different types of competition relationships
   1. Each type of competition relationship should contain different label

## Critical User Journey

1. Firstly, users will see the abstract of the Competition Graph Paper to get an idea of how the competition graph is extracted. Then user can click on the "Try it out" button to try this app
2. Users will fill a form instructing the system which company they want to view. Some recommended tickers are provided, e.g. MSFT, AMZN, AAPL, etc. Users will also tell the system how many layers of relationship to expand, and the number of nodes to view. Then users click on the submit button to view the graph. The frontend will submit users' requests to the backend. If the company ticker entered is found, the backend will return the graph data, and page will be redirected to the graph visualizer. Otherwise, the backend will report an error indicating that the company is not found.. The frontend will display the error message.
3. In the graph visualizer, users can click, select and drag the graph. Users can also zoom in or zoom out. If the clicked node is a company, the frontend will send a request to fetch nodes connected to that company. If successful, the graph node will expand to display the competitors of that company. Otherwise, the frontend does nothing. If the node is already expanded, then all nodes connecting to that node will collapse and become hidden.

<p style="text-align:center"><img src="image/10xGraphWebAppDesignDoc/1686807119816.png"></p>

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

<p style="text-align:center"><img src="image/10xGraphWebAppDesignDoc/1685946694487.png"></p>

## Key UX Design

[Check out the UI](./ux_design/competition-graph-hifi-ui.pptx)

## Frontend Project Structure

```
src
|____api
| |____company-graph.js
| |____form-itemization.js
|____App.vue
|____assets
| |____data
| | |____catelog.txt
| | |____companies.json
| | |____data.json
| | |____dow30_relation_backend.json
| | |____sample.json
| | |____samples_for_front_end.zip
| | |____sample_data
| | | |____AAPL.json
| | | |____AXP.json
| | | |____MSFT.json
| | | |____UNH.json
| | | |____VZ.json
| | | |____WMT.json
| | |____sp500_relation_backend.json
| |____Document.html
| |____Document_files
| |____hl_finance.jpg
| |____logo.png
| |____styles
| | |____bootstrap.min.css
| | |____grid-framework.less
| |____wharton_white.svg
| |____wrds_logo.svg
|____components
| |____CompanyGraph.vue
| |____ItemizationForm.vue
| |____layouts
| | |____Breadcrumb.vue
| | |____Footer.vue
| | |____Header.vue
| |____PaperAbstract.vue
| |____Reader.vue
|____main.js
|____router
| |____index.js
|____views
| |____CompanyRelations.vue
| |____FormReader.vue
| |____Home.vue
| |____SampleForm.vue
```

## Backend Project Structure

```
.
|____app.py
|____blueprints
| |____comp_graph_bp.py
| |____container.py
| |______init__.py
|____container.py
|____data
| |____dow30_relation.json
| |____entity2id.txt
| |____relation2id.txt
| |____sp500_relation.json
| |____train2id.txt
|____entities
| |____graph_dto.py
| |____graph_entities.py
| |______init__.py
|____factory.py
|____repository
| |____comp_graph_repository.py
| |____comp_graph_repository_impl.py
| |____container.py
| |______init__.py
|____services
| |____comp_graph_service.py
| |____comp_graph_service_impl.py
| |____container.py
| |______init__.py
|____utils.py
|______init__.py
```

## DB Design

### Graph Files

DB Choice: 3 Files in Hard Drive

Schema:

- train2id: (e1, e2, rel)
- relation2id: (relation name, id)
- entity2id: (entity, id)
