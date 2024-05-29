[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/1lXY_Wlg)


# Proposal: Financial Market Data Aggregator

## Project Overview

The Financial Market Data Aggregator is designed to deliver a comprehensive platform for real-time and historical insights into the stock market. By consolidating data from diverse sources, the system will facilitate tracking stock prices, analyzing market trends, and making informed investment decisions. Utilizing modern data engineering practices, we will build robust and scalable data pipelines and analytical tools.

## Use Case

The platform will serve investors and financial analysts by providing:

1. **Market Trend Tracking:** Real-time updates on stock prices and market movements.
2. **Historical Analysis:** Insights into historical stock data to identify patterns and trends.
3. **Predictive Analytics:** Machine learning models to predict future stock performance based on historical data.
4. **Visualization:** Tableau dashboards for visualizing market dynamics.

The predictive analytics component will leverage machine learning to forecast daily high and low stock prices, aiding in data-driven investment decisions.

## Data Sources

1. **Dynamic Data Source:** Real-time financial market data from Alpha Vantage API. (https://www.alphavantage.co)
2. **Static Data Source:**
    a. Historical stock market data from Yahoo Finance. (https://help.yahoo.com/kb/SLN2311.html)
    b. Historical data provides up to 10 years of daily historical stock prices and volumes for each stock - nasdaq. (https://www.nasdaq.com/market-activity/quotes/historical)

### Additional Data Source (If time Permits)

- **Economic Indicators:** We may integrate data from the Federal Reserve Economic Data (FRED) API to enrich the analysis with macroeconomic indicators such as interest rates, inflation rates, and GDP growth, providing a more comprehensive market outlook. (https://fred.stlouisfed.org/docs/api/fred/)


_________________


## Components to be Built

### 1. Data Ingestion and Storage

- **APIs and Batch Processing:** We will utilize APIs for real-time data and batch processing for historical data.
- **Data Storage:** Tables will be created in trino wareshouse. The table format will be iceberg.

### 2. Data Pipelines

- **ETL Processes:** We will develop ETL pipelines to extract, transform, and load data. Ensure high data quality and consistency using Apache Airflow for workflow orchestration and dbt for data transformations. We will use Apache Spark to manage high volume of data whereever required.
- **Data Quality Checks:** We will implement data validation with tools like dbt-test or glue data quality.

### 3. Data Modeling

- **Dimensional Modeling:** We will create a dimensional model for optimized analytical queries.
- **Diagramming:** We will attached pdf or image export the target data models archietcture diagram, and data flow diagrams using tools like draw.io or Lucidchart.

### 4. Machine Learning

- **Model Development:** We will develop and train machine learning models (e.g., ARIMA, LSTM) to predict stock prices.
- **Model Deployment:** We will Utilize AWS SageMaker model inference.

### 5. Visualization and Analytics

- **Dashboard Development:** Wwe will create interactive Tableau dashboards to display analytical results.
- **User Interface:** The UI will be limited to Tableau for input criteria and data trend visualizations.

### 6. Recurring Pipeline Jobs

- **Scheduling:** We will use Apache Airflow to automate recurring data pipelines and data quality checks.
- **Monitoring and Alerts:** Implement monitoring and alerting mechanisms to ensure pipeline health and timely data updates.


_________________

### Tentative data model based on Star Schema:

#### Fact Table: `Fact_Stock_Prices`

- **Date_Key (FK)**
- **Stock_Key (FK)**
- **Source_Key (FK)**
- **Open_Price**
- **Close_Price**
- **High_Price**
- **Low_Price**
- **Volume**

#### Dimension Tables

**Dim_Date**

- **Date_Key (PK)**
- **Date**
- **Day**
- **Month**
- **Quarter**
- **Year**
- **Weekday**

**Dim_Stock**

- **Stock_Key (PK)**
- **Stock_Symbol**
- **Company_Name**
- **Sector**
- **Industry**
- **Exchange**

**Dim_Source**

- **Source_Key (PK)**
- **Source_Name**
- **Source_Type (Static/Dynamic)**


_________________


### Visualization of Star Schema

**A visual representation of the star schema:**


```plaintext
              +--------------------+
              |     Dim_Date       |
              +--------------------+
              | Date_Key (PK)      |
              | Date               |
              | Day                |
              | Month              |
              | Quarter            |
              | Year               |
              | Weekday            |
              +--------------------+
                     |
                     |
    +----------------+-----------------+
    |                |                |
    |                |                |
    |                |                |
+--------------------+        +--------------------+  
|   Fact_Stock_Prices|        |   Dim_Stock        |
+--------------------+        +--------------------+
| Date_Key (FK)      |        | Stock_Key (PK)     |
| Stock_Key (FK)     |        | Stock_Symbol       |
| Source_Key (FK)    |        | Company_Name       |
| Open_Price         |        | Sector             |
| Close_Price        |        | Industry           |
| High_Price         |        | Exchange           |
| Low_Price          |        +--------------------+
| Volume             |              
+--------------------+              
                     |                  
                     |                  
                     |                  
                     |                  
              +--------------------+  
              |    Dim_Source      |  
              +--------------------+  
              | Source_Key (PK)    |  
              | Source_Name        |  
              | Source_Type        |  
              +--------------------+  
```

              
_________________


### What each table represents?

1. **Fact Table: Fact_Stock_Prices**
   - Central table containing quantitative data about stock prices and volumes.
   - Contains foreign keys (`Date_Key`, `Source_Key` and `Stock_Key`) linking to the corresponding dimension tables.

2. **Dim_Date**
   - Dimension table containing date-related attributes, allowing for time-based analyses.
   - `Date_Key` is the primary key.

3. **Dim_Stock**
   - Dimension table containing stock-related attributes such as the stock symbol, company name, sector, industry, and exchange.
   - `Stock_Key` is the primary key.

4. **Dim_Source**
   - Dimension table containing information about the data sources (e.g., Alpha Vantage for dynamic data, Yahoo Finance for static data).
   - `Source_Key` is the primary key.


_________________


### What is Usage of each table in the model

- **Fact_Stock_Prices** table allows for detailed analysis of stock prices and volumes, linked by date and stock.
- **Dim_Date** table enables slicing and dicing data by different time periods.
- **Dim_Stock** table allows for analysis based on stock attributes such as sector or industry.
- **Dim_Source** By including Source_Key in the Fact_Stock_Prices table, each fact record can be associated with its respective data source. This helps in:

    Data Quality Analysis: Understanding and validating the consistency and reliability of data from different sources.
    Source-Specific Analysis: Comparing data trends from different sources.
    Troubleshooting: Identifying issues with data ingestion from specific sources.


_________________


### Machine Learning Model

#### Overview

Details of developing and deploying machine learning models to predict stock prices. Using the fact-dimensional model, the machine learning pipeline will leverage historical and real-time data to train models and make predictions. The following are the detailed steps outlining the process for building this ML Model.

#### Steps to Build Machine Learning model

1. **Data Extraction:**
   - Extract data from the Trino data warehouse, particularly from the `Fact_Stock_Prices` table, along with relevant attributes from the `Dim_Date` and `Dim_Stock` tables.
   - Filter and aggregate the data based on the prediction requirements (e.g., predicting next day's high/low prices).

2. **Data Preparation:**
   - Preprocess the data to handle missing values, outliers, and ensure data consistency.
   - Create additional features (e.g., moving averages, volatility) based on historical stock prices.
   - Normalize/standardize the data for better model performance.

3. **Feature Engineering:**
   - Generate time-series features such as lagged values, rolling means, and differences.
   - Incorporate external features if needed, such as macroeconomic indicators or news sentiment scores.

4. **Model Development:**
   - Split the data into training, validation, and test sets.
   - Train machine learning models (e.g., ARIMA, LSTM, XGBoost) on the training set.
   - Perform hyperparameter tuning and cross-validation to optimize model performance.
   - Evaluate model performance using appropriate metrics (e.g., RMSE, MAE).

5. **Model Deployment:**
   - Deploy the trained model using a platform like AWS SageMaker.
   - Set up an inference pipeline to make real-time predictions using incoming data.
   - Integrate the predictions back into the data warehouse or a dedicated prediction table for visualization and analysis.

6. **Visualization and Analytics:**
   - Use Tableau to visualize model predictions and compare them with actual stock prices.
   - Create dashboards to display prediction accuracy, model performance metrics, and trends.
  

_________________


### Prediction Table Design for Visualization

The prediction table will store the results of the machine learning model predictions, which can then be visualized in a Tableau dashboard. This table will include the predicted stock prices along with the actual prices for comparison, as well as relevant metadata to facilitate detailed analysis and visualization.

#### Table: `Fact_Predicted_Stock_Prices`

**Columns:**

1. **Prediction_ID (PK)** - A unique identifier for each prediction record.
2. **Date_Key (FK)** - Foreign key referencing the `Dim_Date` table.
3. **Stock_Key (FK)** - Foreign key referencing the `Dim_Stock` table.
4. **Source_Key (FK)** - Foreign key referencing the `Dim_Source` table.
5. **Predicted_Date** - The date for which the prediction is made.
6. **Predicted_Open_Price** - The predicted opening price of the stock.
7. **Predicted_Close_Price** - The predicted closing price of the stock.
8. **Predicted_High_Price** - The predicted highest price of the stock.
9. **Predicted_Low_Price** - The predicted lowest price of the stock.
10. **Actual_Open_Price** - The actual opening price of the stock (if available).
11. **Actual_Close_Price** - The actual closing price of the stock (if available).
12. **Actual_High_Price** - The actual highest price of the stock (if available).
13. **Actual_Low_Price** - The actual lowest price of the stock (if available).
14. **Prediction_Model** - The model used for making the prediction (e.g., ARIMA, LSTM).
15. **Prediction_Timestamp** - The timestamp when the prediction was made.

### Visualization Table Schema

```plaintext
Fact_Predicted_Stock_Prices
+------------------------+--------------+
| Column                 | Data Type    |
+------------------------+--------------+
| Prediction_ID (PK)     | INT          |
| Date_Key (FK)          | INT          |
| Stock_Key (FK)         | INT          |
| Source_Key (FK)        | INT          |
| Predicted_Date         | DATE         |
| Predicted_Open_Price   | FLOAT        |
| Predicted_Close_Price  | FLOAT        |
| Predicted_High_Price   | FLOAT        |
| Predicted_Low_Price    | FLOAT        |
| Actual_Open_Price      | FLOAT        |
| Actual_Close_Price     | FLOAT        |
| Actual_High_Price      | FLOAT        |
| Actual_Low_Price       | FLOAT        |
| Prediction_Model       | VARCHAR      |
| Prediction_Timestamp   | TIMESTAMP    |
+------------------------+--------------+
```

### Metadata

1. **Prediction_ID:** A unique identifier for each prediction record, ensuring each entry can be uniquely referenced.
2. **Date_Key:** Links to the `Dim_Date` table, allowing for time-based aggregation and filtering.
3. **Stock_Key:** Links to the `Dim_Stock` table, providing contextual information about the stock.
4. **Source_Key:** Links to the `Dim_Source` table, indicating the source of the data.
5. **Predicted_Date:** The specific date for which the prediction is made, crucial for time-series analysis.
6. **Predicted_Open_Price:** The predicted opening price for the stock.
7. **Predicted_Close_Price:** The predicted closing price for the stock.
8. **Predicted_High_Price:** The predicted highest price for the stock during the day.
9. **Predicted_Low_Price:** The predicted lowest price for the stock during the day.
10. **Actual_Open_Price:** The actual opening price for the stock (if known at the time of visualization).
11. **Actual_Close_Price:** The actual closing price for the stock (if known at the time of visualization).
12. **Actual_High_Price:** The actual highest price for the stock during the day (if known at the time of visualization).
13. **Actual_Low_Price:** The actual lowest price for the stock during the day (if known at the time of visualization).
14. **Prediction_Model:** The model used to make the prediction, which helps in assessing the performance of different models.
15. **Prediction_Timestamp:** The time when the prediction was generated, useful for tracking and audit purposes.

### Visualization to be done on Prediction Table

In Tableau, this table can be used to create various visualizations, such as:

1. **Comparison Charts:** Actual vs. predicted prices (e.g., line charts or bar charts) for different stocks over time.
2. **Error Analysis:** Visualizing prediction errors to identify patterns or anomalies.

