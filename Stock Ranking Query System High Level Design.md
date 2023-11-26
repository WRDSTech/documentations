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
  * 

##### Trade

1. Volume (1 day, 5 day, 20 day)
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

## Database Design

Specify the choice of database and how data should be modelled.

### Table Design

## Stock Ranking Factors Specification

Specify what factors should be supported and how to retrieve and calculate them.

## High Level UI Design

## Implementation Story

## References and Additional Information
