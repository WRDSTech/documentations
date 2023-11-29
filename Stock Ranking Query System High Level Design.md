# WRDS Stock Ranking Query System High-Level Design

## Design Objective

The goal of this document is to provide detailed guidance and  specification for the development of the WRDS Stock Ranking Query System. WRDS Stock Ranking Query System is a subsystem of the WRDS Stock Ranking System that consists of a data ingestion system and a query system. The design objective of whole system is to recommend stocks and to help users forming their own strategies based on a set of stock ranking factors. While the data ingestion system handles data ingestion and calculation, this subsystem is primary responsible for providing views and queries of stock rankings to users. This document includes many aspects of the system, including architecture, data models, factor specification,

## Background

The Stock Ranking system, hereinafter referred to as "the System," represents a crucial tool for investors and financial professionals seeking to make data-driven investment decisions. The System is designed to efficiently collect, process, and analyze a vast amount of financial data, including historical stock prices, market indicators, and various financial metrics. This background section provides an overview of the motivations and context driving the development of the System.

### Motivation

In today's rapidly evolving financial markets, the ability to make informed investment decisions is paramount. Access to comprehensive and accurate financial data is essential for investors to identify trends, assess risk, and seize opportunities. Traditional data processing and analysis methods often struggle to handle the ever-increasing volume of financial data generated daily. As a result, there is a growing need for a robust and high-performance financial analysis system that can efficiently process and analyze large datasets to support investment strategies.

### **Context**

The System is being developed in response to the challenges and opportunities presented by the financial industry. With the advent of online trading platforms, algorithmic trading, and the democratization of financial information, investors and financial professionals are increasingly relying on technology to gain a competitive edge in the market. However, these technological advancements also pose the challenge of handling massive datasets while ensuring data accuracy, integrity, and availability.

To address these challenges, the System leverages the Hadoop ecosystem  like Hive and Spark for OLAP workloads. The Hadoop ecosystem provides mature component to support data intensive usecases, allowing the System to provide users with timely and actionable insights for investment decision-making.

## Overview

This document aims to analyze the platform and give a tentative breakdown of each of its components. It starts with analyzing problem requirements and then breaks down the platform step by step. A couple of solutions will be discussed for sub-components, which will be analyzed later in more specific designs. Finally, a road map will be provided for this project.

## Requirement

### Functional Requirement

#### Stock ranking

* Users can select a range of filter conditions and ranking factors to form a ranked list of stocks
  * Users can choose the date of data to rank stocks
    * The date is set to the latest trading date by default
    * Users can also choose a past date
      * The future return will be listed in the result
        * 1-day return
        * 5-day return
        * 20-day return
  * Users can adjust the threshold values of filter conditions
    * A pool of stocks will be filtered and selected from these filter conditions
    * Then this pool of stocks will be ranked based on the weighted sum of ranking scores
  * Users can adjust the weight of each ranking factors
  * Users can view the ranking result based on the ranking scores
    * the ranking score is ranged from 0 - 100
    * Users can select the direction of the ranking
      * High to Low
      * Low to High
    * the ranking score is calculated by a weighted sum of all ranking factors
      * the value of ranked factors of each stock is converted to rank score
        * rank score = (n_stocks - rank of stock + 1) / n_stocks *100
      * final rank score = weight_1 * rank_score_on_factor_1 + weight_2 * rank_score_on_factor_2 + ... + weight_n * rank_score_on_factor_n
      * If a stock has no valid ranking factor value, it will get a rank score of 0
* Users can adjust the number of stocks to display in the final result
  * default to 20
  * some quick options
    * 10
    * 20
    * 50
    * 100
    * no limit
* Users can view and export the ranking result as excel file
  * The result includes the following items
    * stock ticker
    * stock name
    * close price
    * volume
    * weighted sum of ranking score
    * value of rank factor 1
    * value of rank factor n (if any)
    * ranking score of factor 1
    * ranking score of factor n (if any)

#### Factor Customization (Advanced Requirement)

This requirement is planned to be implemented in multiple stages as it is quite complicated.

* Users can customize their own factors based on system built-in factors and data
  * Stage 0: Proof of concept, internal configuration only
    * provide a internal interface to manage factors
    * Admin can write computation script (python / sql) to generate factors
  * Stage I: Only support factor building with basic arithmetic operations
    * Add, Subtract, Multiply, Division
    * If factors do not have the same time period, all of them must be downscaled to the one with the lowest frequency
  * Stage II: Support predefined functions
    * Such as:
      * Mean
      * Max
      * Min
      * Zscore
      * Standardize

#### Representative User-Journey

A user wants to get the top 20 stocks based on the Quick Ratio, ROE and EV/EBITDA in order to assess stocks in terms of debt repayability, profitability, and valuation. The user want to exclude stocks with market cap under $100 million. He or she select "market cap" as the filter condition, select the condition to be "larger or equal to", and enter $100 million. Then the user select "Quick Ratio, ROE and EV/EBITDA" as ranking factors. Equal weights are given to all factors. All factors are set to rank in descending order. When the user submit the request, the ranking will be calculated by the weighted sum of all ranking scores of the chosen factors. Then the user can view the result includingh tickers, names, ranking scores, and factor scores. The user can also export the result as excel.

#### Stock Ranking Factors

##### Fundamental

* Valuation
  * P/E
    * yearly
    * TTM
  * P/B
  * EV / EBITDA
  * PEG
  * EP
  * BP
* Debt Repayability
  * Quick Ratio
  * Current Ratio
  * Debt / Asset
* Capital Structure
  * Equity / Asset
  * Fixed asset / Asset
* Profitablility
  * Revenue / Income
  * Net Income / Revenue
  * Gross Income / Revenue
  * Net Income / Asset
  * Net Income / Equity
  * ROIC
* Operation Efficiency
  * Account Receivable Turnover Rate
  * Current asset turnover rate
  * Inventory turnover rate
* Growth
  * Revenue growth
  * Net income growth
  * Gross income growth
  * Net asset growth
  * Net Income 3-year CAGR
  * Net Income 5-year CAGR
* Per share metrics
  * Earning per share
  * dividend per share
  * Free Cash Flow per share

##### Trade

1. Price and Volume (OHLCV)
2. MA (1 day, 5 day, 20 day)
3. Momentum (1 day, 5 day, 20 day)
4. turnover rate (1 day, 5 day, 20 day)
5. market cap
6. macd dif (1 day, 5 day, 20 day)
7. volatility (1 day, 5 day, 20 day)

### Non-Functional Requirement

* The system should be adaptable and compatible with multiple existing strategies/data processing workflow.
* It should be scalable, such that a massive amount of data could be injected and processed.
* The platform should be reliable, client should have no problem accessing the ranking result.

#### Performance

* For ranking queries, we have around 8000 companies' data and 2 to 150 combined factors in general. We’d like to target 1,000 query requests per second (QPS) in the first stage.
* When requests arrive, we aim to get it around 100ms, either by smart computation or caching.

## System Architecture

### High-Level Overview

![1701180869342](image/StockRankingQuerySystemHighLevelDesign/1701180869342.png)

The Stock Ranking Query System consists of four microservices:

* Factor Manager
* Stock Ranker
* Stock Pool Manager
* Portfolio Manage

**Factor Manager:**

- Manages a dynamic catalog of stock ranking and filtering factors.
- Allows users to create custom factors using a formula editor that supports arithmetic operations and references to other factors.
- Provides a user interface for browsing and searching available factors.
- Ensures the integrity of factor definitions and dependencies.

**Stock Ranker:**

- Receives queries from users to rank stocks based on selected factors.
- Utilizes a high-performance computation engine to process large volumes of stock data.
- Offers various ranking algorithms, including weighted scoring and machine learning models.
- Supports real-time and batch processing modes for different use cases.

**Stock Pool Manager:**

- Curates and manages a collection of stock pools, such as blue-chip stocks, tech sector stocks, or value stocks.
- Allows users to define their own stock pools based on specific criteria or filters.
- Integrates with external data sources to update stock pools based on market changes.
- Provides APIs for other system components to retrieve and subscribe to stock pool updates.

**Portfolio Analyzer:**

- Tracks the historical and real-time performance of stock rankings and user portfolios.
- Offers analytical tools to compare the performance against benchmarks or indices.
- Allows users to set up alerts based on portfolio performance metrics.
- Generates detailed reports with insights into portfolio diversification, risk, and returns.

**Data Digestion System:**

- Acts as a backbone for the system, ingesting and normalizing data from various stock exchanges and financial data providers.
- Ensures data quality and consistency through validation, cleaning, and transformation processes.
- Provides a scalable storage solution to handle the vast amounts of incoming financial data.
- Facilitates data accessibility for the microservices through a well-defined API.

**Infrastructure (Infra):**

- Built on a robust cloud infrastructure to ensure scalability, reliability, and security.
- Implements containerization and orchestration for microservices deployment and management.
- Employs a combination of relational and NoSQL databases optimized for different data access patterns.
- Features a comprehensive monitoring and logging system to track system health and performance.

### System Component Design

![1701181323446](image/StockRankingQuerySystemHighLevelDesign/1701181323446.png)


**Stock Ranking Web App:**

- **Factor Panel:** Allows users to browse, select, and manage the factors used in stock ranking. Users can create custom factors by defining formulas or expressions.
- **Rank Condition Selector:** Enables users to set conditions and criteria for stock ranking, like thresholds or specific factor weightings.
- **Stock Pool Selector:** Provides a user interface for selecting predefined pools of stocks or for creating custom pools based on user-defined criteria.
- **Ranking Panel:** Displays the results of stock rankings based on selected factors and conditions. Users can sort, filter, and analyze the ranking results.
- **Portfolio Tracker:** Tracks the performance of user portfolios over time and provides insights into gains, losses, and other relevant metrics.
- **Account Management:** Manages user accounts, including authentication, authorization, preferences, and settings.

**API Gateway:**

- Serves as an entry point for the web application, routing requests to the appropriate services in the application layer and aggregating the results for the web app.

**Application Layer:**

- **Factor Manager:** Manages the lifecycle of ranking factors, including creation, update, and deletion of factor definitions.
- **Stock Ranker:** Processes ranking requests by applying factors and conditions to generate a ranked list of stocks.
- **Stock Pool Manager:** Manages the different pools of stocks, including their creation, update, and deletion based on defined criteria.
- **Portfolio Manager:** Handles operations related to portfolio management, including tracking performance and managing portfolio compositions.

**Domains:**

- **Factor Domain:** Contains the business logic related to ranking factors and their management.
  - Aggregate:
    - FactorSet
      - Represents the whole set of factors exposed to users.
        - Factors in quantitative finance are variables that capture specific sources of risk and return in investment portfolios. They are used to explain performance and to design strategies that capitalize on these risk-return relationships.
      - supported operations
        - create new factor
        - view factors
        - update factors
        - delete factors
        - factor state management
          - enable update
          - pause update
  - Domain Objects
    - Factor
  - Value Objects
    - FactorValue
    - FactorState
- **Ranker Domain:** Encapsulates the logic for the ranking process, including the application of algorithms and factor weightings.
- **Stock Domain:** Represents the domain logic for managing stock-related information, such as stock details, price history, and metadata.
- **Portfolio Domain:** Governs the rules and operations related to portfolio management, performance tracking, and analytics.

**Repositories:**

- **Factor Repository:** Responsible for persisting factor data and providing access to factor entities.
- **Ranker Repository:** Handles persistence and retrieval of ranking computations and results.
- **Stock Repository:** Manages the storage and retrieval of stock data, including real-time and historical data.
- **Portfolio Repository:** Maintains portfolio data, including holdings, transaction history, and performance metrics.

**Infrastructure:**

- **Time Series DB:** Stores and manages time-series data like stock prices, volume, and other temporal data.
- **Relational DB:** Manages structured data, such as user information, factor definitions, and portfolio compositions.
- **Redis:** Provides in-memory data storage for fast access to frequently used data like session states, cache, and temporary computations.
- **RabbitMQ:** Handles message queuing for asynchronous processing and communication between different services and components.

**Query System External Interface Manager:**

- Manages the interfaces between the application and external query systems, ensuring the seamless flow of data and requests.

**Stock Ranking Data Digestion System:**

- Processes and normalizes raw stock data from various external sources, preparing it for consumption by the application's services.




## Database Design

The query system chooses **PostgreSQL** and its time-series supporting variant **TimescaleDB** as primary databases. Trade-offs and reasons behind this decision will be discussed in the database trade-off section.

### Table Design

how to design tables

### Trade-offs

Compare Postgres / TimescaleDB with other databases

1. [Vanilla Postgres vs TimeScaleDB](https://medium.com/timescale/timescaledb-vs-6a696248104e)

## Stock Ranking Factors Specification

Specify what factors should be supported and how to retrieve and calculate them.

## High Level UI Design

## Implementation Story

## References and Additional Information


1. [Adding TimescaleDB to Postgres](https://bobcares.com/blog/add-timescale-to-postgresql/)
