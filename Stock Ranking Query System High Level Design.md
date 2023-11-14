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

..

## Requirement

### Functional Requirement

...

#### Primary Usecases

##### Ranking Factor Query & Selection

##### Stock Ranking

##### Export Result as Excel

#### Stock Ranking Factors

##### Fundamental

1. ROE
2. Gross Income

##### Trade

1. Volume
2. MA5
3. Momentum

### Non-Functional Requirement

...

#### Performance

...

#### Extensibility

...

## System Architecture

## Database Design

Specify the choice of database and how data should be modelled.

### Table Design

## Stock Ranking Factors Specification

Specify what factors should be supported and how to retrieve and calculate them.

## High Level UI Design

## Implementation Story

## References and Additional Information
