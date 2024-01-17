# Stock Factor Management Service Design Document

## 1. Introduction

### Purpose

The primary purpose of this system is to manage stock factor metadata, calculate factors based on stock market data, and regularly update these factors. This system serves as a core component in a broader stock trading and ranking framework, enabling dynamic and accurate assessment of stock performance.

### Scope

The document covers the integration and data flow between the data ingestion system, factor management system, and stock ranking system, with a focus on the underlying infrastructure.

### Audience

This document is intended for system architects, developers, and stakeholders involved in the development and implementation of the stock factor management service.

## 2. System Overview

This section presents an overview of the independent systems — Data Ingestion System, Factor Management System, and Stock Ranking System — and their collaborative functioning within the broader stock recommendation system.

### Independent Systems Description

1. **Data Ingestion System** :

* **Functionality** : Primarily responsible for ingesting raw stock market data from various sources, predominantly in CSV format.
* **Autonomy** : Operates independently, focusing on accurate and efficient data collection and preliminary processing.
* **Technology** : Utilizes Hadoop for data processing and Hive for initial data storage.
* **Role in Collaboration** : Acts as the primary data provider, ensuring the availability of up-to-date and comprehensive market data for subsequent analysis by other systems.

2. **Factor Management System** :

* **Purpose** : Manages and calculates stock factors, pivotal in analyzing stock performance.
* **Independence** : Functions autonomously in managing factor metadata and executing complex calculations.
* **Database Use** : Employs Postgres with the TimescaleDB extension for efficient handling and storage of time-series data related to stock factors.
* **Collaborative Aspect** : Receives raw data from the Data Ingestion System and, after processing, makes factor data available for the Stock Ranking System, facilitating a seamless flow of information.

3. **Stock Ranking System** :

* **Primary Function** : Uses calculated factors to rank stocks, aiding in the generation of stock recommendations.
* **Operational Independence** : Works independently to apply ranking algorithms and generate insights.
* **Data Integration** : Integrates factor data from the Factor Management System to perform stock rankings.
* **Contribution to the Broader System** : Plays a critical role in providing actionable stock recommendations within the stock recommendation system.

### Collaborative Framework within the Stock Recommendation System

* **Integrated Workflow** : While each system operates independently, they are interconnected through a well-defined data flow, ensuring efficient collaboration within the stock recommendation system.
* **Data Flow Dynamics** :
* The Data Ingestion System serves as the starting point, gathering and preprocessing stock market data.
* This data is then utilized by the Factor Management System for factor calculation and management.
* The Stock Ranking System, in turn, uses these factors to rank stocks, influencing stock recommendation strategies.

### Infrastructure Overview

* **Hadoop and Hive** : Core to the Data Ingestion System for managing large datasets.
* **Apache Airflow** : Employed in the Data Ingestion System for scheduling and managing data-related workflows.
* **Postgres with TimescaleDB** : Utilized by both the Factor Management and Stock Ranking Systems for efficient data storage and retrieval, particularly suited for time-series data.

## 3. Data Flow and Integration

This section describes the process of data flow within the stock factor management service, focusing on the calculation and updating of foundational and custom factors.

### Data Flow Overview

1. **Scheduled Workflow Initiation** :

* Apache Airflow triggers the update workflow as per a predefined schedule, ensuring timely factor updates.
* This scheduling is crucial for maintaining up-to-date factor calculations reflective of the latest market data.

1. **Data Request and Retrieval** :

* The Factor Management System, upon initiation, requests necessary raw data from the Data Ingestion System.
* This request is based on factor metadata, which specifies data dependencies, responsible modules for data calculation, and calculation parameters.

1. **Factor Calculation** :

* Received data is sent to the computation engine within the Factor Management System.
* Here, both foundational and custom factors are calculated according to their defined methodologies.

1. **Database Update** :

* Upon successful calculation, the Factor Management System updates the factors in the database, making them available for downstream processes like stock ranking.

### Foundational Factors Calculation

Foundational factors are derived directly from raw data or calculated using built-in libraries of the Factor Management System. Examples include:

* **Moving Averages (MA5, MA10)** : Calculated as the average stock price over 5 and 10 days, respectively.
* **MACD, MFI, RSI** : Indicators calculated using standard financial formulas to assess momentum, money flow, and strength of stock movements.
* **Return Metrics (Ret5, Ret20)** : Represent the stock's performance over 5 and 20 days.
* **Financial Ratios (ROE, PB, PE_ttm, EPS, Quick Ratio, Current Ratio, ROIC)** : Derived from the company's financial statements, representing various aspects of financial health and performance.

### Custom Factors Calculation

* **User-Defined Expressions** : Custom factors are defined by users using expressions that combine functions, operators, and foundational factors.
* **Dynamic Calculation** : These factors are dynamically calculated based on user-defined criteria, providing tailored insights into stock performance.
* **Example** : A custom factor could be a weighted combination of 'MA10' and 'RSI', defined by a user to gauge a specific market trend.

### Integration of Foundational and Custom Factors

* **Supportive Structure** : Foundational factors form the base for the calculation of more complex custom factors.
* **Data Flow** : Once foundational factors are updated in the database, they become immediately available for use in custom factor calculations.

### High-Level Workflow of Factor Calculation and Update

1. **Trigger** : The workflow begins with Airflow triggering the scheduled update.
2. **Data Retrieval** : Factor Management System requests and retrieves raw data.
3. **Computation** : Calculation of foundational and then custom factors.
4. **Update** : Factors are updated in the database for use in stock ranking and other analyses.

* **CSV File Handling** : CSV files containing stock data are ingested through Hadoop and stored in Hive.
* **Workflow Management** : Apache Airflow orchestrates the workflow of data processing.

### Data Flow Diagram

* *Note: A comprehensive data flow diagram will be added in subsequent revisions.*

## 4. Accessibility Considerations

### Accessible Design Principles

* **User Interface** : If applicable, the UI will adhere to WCAG guidelines to be accessible to users with disabilities.
* **Data Presentation** : Data visualizations will include alternative text and be compatible with screen readers.

## 5. Database Schema

### Schema for Factor Management

* *Details of the database schema for storing factors and metadata will be included here.*

### Schema for Stock Ranking

* *Details of the database schema for storing stock ranking results will be included here.*

## 6. Security and Compliance

### Data Security

* **Protection Measures** : Encryption, access controls, and other security measures will be outlined.

### Regulatory Compliance

* **Financial Data Handling** : Compliance with relevant financial regulations will be detailed.

## 7. Testing and Validation

### Testing Strategy

* **Integration Testing** : Ensuring seamless integration between different components.
* **Functionality Testing** : Verifying the accuracy of factor calculations and ranking algorithms.

### Validation Methods

* **Algorithm Validation** : Techniques for validating the efficacy of the stock ranking algorithm.

## 8. Scalability and Performance

### Handling Large Data Volumes

* Discussion on scalability and performance optimization for large data sets.

### Performance Metrics

* Definition of key performance indicators for the system.

## 9. Conclusion

### Summary

* Recap of the design principles and key components of the stock factor management service.

### Future Considerations

* Potential enhancements and scalability options.

## 10. Appendices

### Glossary

* Definitions of technical terms used in the document.

### References

* Citations of external sources or documents.

## 11. Revision History

### Document Tracking

* Log of updates and revisions to the document.
