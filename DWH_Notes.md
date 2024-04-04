# DWH part 1

## What is Data Warehouse - Business View

- You are the head of ecommerce data infrastructure
- Business day to day work
  - **Customers** should be able to find goods and create new orders
  - **Inventory** Staff should be able to stock, retrieve and re-order goods
  - **Delivery** Staff should be able to pickup and deliver goods
  - **HR** should be able to assess performance of staff
  - **Marketing** should be able to attract new custmners
  - **Management** should be able to Inonitor sales groysth
- Questions for you
  - Can I build a databases to support these activities?
  - Are all of tlle above of the saine nature?

## Operational vs Analytical Business Processes

<div style="clear: both;">
  <div style="float: left; margin-right: 1em;">
    <img src="attachments\operational_process.png" width="125">
  </div>
  <div>
    <p>- Able to find products and make orders </br>
- Added new products and check stocks (Inventory Staff)</br>
- Pick up & Deliver Products (Delivery staff)</p>
  </div>
</div>

<div style="clear: both;">
  <div style="float: left; margin-right: 1em;">
    <img src="attachments\analytical process.png" width="125">
  </div>
  <div>
    <p>- Check sales performance (for HR)</br>
- See effect of different sales channels (Marketing)</br>
- Sales growth (Nlanagelllent)</br>
- Understand Customer behavior (Data Team)</br></p>
  </div>
</div>

## Operational Databases vs DWH (Analytical database)

<img src="attachments\database_OLAP_OLTP.png" width=400>

## Pros of operational databases

- Excellent for operations
- No redundancy
- Fast insert and update

## Cons of Operational databases

- Too slow for analytics
- Too may joins
- Hard to understand


## Data warehouse

### Definition

In computing, a data warehouse, also known as an enterprise data warehouse, is a system used for reporting and data analysis and is considered a core component of business intelligence. DWs are central repositories of integrated data from one or more disparate sources.

<img src="attachments\DW_techical_view.png" width="700">

### Dimentional model is designed to
1. make is easy for business users to work with the data
2. improve analytical queries performance

### Facts & Dimension
**Fact tables (Numeric & Additive)**
- Record business events, like an order, a phone call, a book review
- Fact tables columns record events recorded in quantifiable metrics like quantity of an item, duration of a call, book rating
 
**Dimensional tables**
- Record the context of the business events, e.g. who, what, where, why, etc...
- Dilliension tables coltlillns contain attributes like the store at which an item is purchased, or
the customer who made the call, etc...

- Example of Dimensions:
  - Data & Time are always dimension
  - Physical Location and their attributes
  - Human Roles
  - Products

[datawarehouse-schema-models-kumail-raza](https://www.linkedin.com/pulse/datawarehouse-schema-models-kumail-raza/)

### **Star schema**

<img src="attachments\starSchema.jpeg" alt='Star schema' width="600">


### snowflake schema (Operational datastore)

<img src="attachments\snowflakeschema.jpeg" alt='snowflake schema'>