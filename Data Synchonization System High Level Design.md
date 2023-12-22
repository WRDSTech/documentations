# Data Synchronization System

## Design Objective

The design objective of this system is to build a data synchronization system that synchronizes financial data from some data providers. The goal is to have a periodic and automatic data synchronization mechanism to ease the burden of manually pulling data from these data providers. Since each data provider has their own set of APIs, this system must be flexible enough and provide configurations to adapt to the differences. The localized data should also be stored in various formats: files (such as CSV, excel, parquet) or databases (Mysql, Postgres, Clickhouse, Dolphindb, MongoDB, etc.).

## Background and Requirements

The project serves as the first step towards building an in-house quantitative trading system. It should provide stable and periodic data synchronization with high performance. Rust is considered as the implementation language since it is safe and fast.

The system should adapt to various data providers. Each data provider has a set of data APIs, which can be requested via HTTP request, Websocket, or file downloads. They also limit the frequency and the amount of data per request, so first-time synchronization requires careful planning. For each data API, the synchronization process should be divided into many tasks which contain the address, request method, and parameters (such as date, size, page number, etc). The data size and the request rate should not exceed the limit of the data provider. If the limit is exceeded, the task should be paused or canceled until the limit is released. Most of the time, the task only needs to be paused for a few minutes. Some special data APIs require users to wait for the next day. In this case, the progress must be saved and to be continued.

Each data API has its own set of parameters and requirements. The system should support automatic or manual configuration to adapt to these apps. It will try to parse the API documentation page to fill most of the details but allow users to configure those not covered by the system.

Most of the data are updated daily so the system should create plans to run synchronization tasks on time. Some data are updated and streamed so the system needs to support Websocket streaming. Users can manually add, start, pause, or retry sync plans.

Logging and reporting features are also required to help users understand the result and the status of each synchronization iteration. Some key information could include the success rate, the amount of time taken, the size of data synchronized, etc. Critical errors must be reported to help users fix them.

## Requirement

#### 1. Modular Design for Data Provider Integration

* **Provider Abstraction:** Implement a trait (interface) that each data provider module must fulfill. This trait should include methods for handling API requests, handling rate limits, parsing responses, and error handling.
* **Configurable Parameters:** Each provider module should be able to be configured with its own set of API endpoints, request parameters, rate limits, and data formats.

#### 2. Task Management and Synchronization Logic

* **Task Scheduler:** Develop a system to schedule and manage data synchronization tasks. This might involve a cron-like scheduler for periodic tasks and a more dynamic scheduler for streaming data.
* **Concurrency and Rate Limit Handling:** Utilize Rust's asynchronous programming capabilities to manage multiple tasks concurrently. Implement logic to handle API rate limits, pausing, and resuming tasks as needed.

#### 3. Data Storage and Format Management

* **Storage Abstraction:** Create a storage abstraction layer to handle different storage formats (CSV, Excel, Parquet) and databases (MySQL, Postgres, ClickHouse, etc.). This layer should provide a uniform API for storing data regardless of the underlying storage mechanism.
* **Data Serialization/Deserialization:** Implement or utilize existing Rust libraries for converting data between different formats (e.g., Serde for JSON serialization/deserialization).

#### 4. Configuration and User Interface

* **API Documentation Parsing:** If feasible, implement a system to semi-automatically parse API documentation and fill in configuration details. This can be a complex task depending on the consistency and format of the documentation.
* **User Configuration Interface:** Provide a user-friendly interface for configuring the synchronization tasks, which might include a command-line interface (CLI) or even a simple GUI.

#### 5. Logging, Reporting, and Error Handling

* **Logging System:** Implement a comprehensive logging system to track the operations, successes, and failures of the synchronization tasks.
* **Error Reporting:** Develop a mechanism for reporting critical errors to users, possibly through email alerts or a dashboard.

#### 6. Implementation Considerations in Rust

* **Safety and Performance:** Leverage Rust's memory safety and concurrency features to ensure high performance and reliability.
* **External Crates:** Utilize external crates for HTTP requests (e.g., `reqwest`), WebSockets (e.g., `tokio-tungstenite`), task scheduling (e.g., `tokio`), and database connectors.

### Representative User Journey
