# Breweries Case  

## Objective  
The goal of this test is to assess your skills in consuming data from an API, transforming, and persisting it into a data lake following the medallion architecture with three layers: raw data, curated data partitioned by location, and an analytical aggregated layer.  

## Technologies Used  
For the implementation of the solution, **Python** and **PySpark** were employed for data processing, with **Databricks** being used to execute the ELT (Extract, Load and Tranform) process in the Medallion layers (bronze, silver, and gold). Pipeline orchestration was handled through **Azure Data Factory** (ADF).  

## Architecture and Design  

### Bronze Layer  
**Notebook:** `0_ntbk_breweries_to_bronze`  
**Function:** The bronze layer stores data in the format closest to the original structure provided by the Open Brewery DB API. This layer is responsible for persisting raw data, preserving it for auditing and future reference, making it ideal for historical queries or unstructured analysis. It is worth mentioning that this stage includes basic exception handling for failed requests using the `raise exception` feature.  
**Storage Format:** Data is stored in **JSON** format.  
**Writing Mode:** For simplicity, the data is overwritten at each execution as the API does not require incremental control or complex changes.

### Silver Layer  
**Notebook:** `1_ntbk_bronze_to_silver`  
**Function:** In the silver layer, data undergoes a transformation where it is converted into a columnar format, and schema application takes place. Additionally, data is partitioned by state (location field), which facilitates and optimizes queries specific to geographic locations.  
**Storage Format:** Data is saved in **Delta** format, ensuring performance and data integrity.  
**Writing Mode:** As in the silver layer, the data is overwritten, maintaining simplicity given the nature of the API and avoiding unnecessary duplications.

### Gold Layer  
**Notebook:** `2_ntbk_silver_to_gold`  
**Function:** The gold layer aggregates the data into an optimized model for analysis. In this layer, an aggregated view is created showing the number of breweries by type and location.  
**Storage Format:** Data is also saved in **Delta** format, ensuring consistency across the layers.  
**Writing Mode:** At each execution, the data is updated, reflecting the latest state and providing an up-to-date aggregated view.  

**Note:** The functional programming paradigm was chosen to prioritize development speed, code simplicity, and easier future maintenance, as it avoids the complexity of managing objects and state.  

### Reusable Functions - lib_utils Notebook  
Notebook: `lib_utils`  
Function: This notebook contains utility functions used across the other notebooks for common tasks, such as data transformations, API handling, and logging. Functions in this notebook uses "annotations" and "code strings" in order to ensure that the functions are well-documented, which aids in both development and future maintenance. This approach centralizes reusable code, minimizing duplication and ensuring consistency across the pipeline.  

### Orchestration with Azure Data Factory (ADF)  
ADF is used to orchestrate the sequential execution of the Databricks notebooks. Each notebook represents a layer in the pipeline, with dependencies configured to ensure that each stage starts only after the successful completion of the previous one.  
ADF also manages timeout policies, retries, and failure monitoring, ensuring that the pipeline maintains a reliable and resilient flow.  
The full pipeline configuration can be found in the file: `adf_elt_pipeline_breweries.json`

## Containerization  
Containerization was not used in this project due to the choice of tools **Azure Data Factory (ADF)** and **Databricks** for pipeline implementation. These tools eliminate the need for manual container orchestration and management, as they provide a managed infrastructure that integrates natively with Spark and Hadoop.  
With Databricks, there is no need to manually instantiate a Hadoop/Spark cluster or configure components such as **YARN**. Databricks provides scalable and configurable clusters optimized for distributed processing, while ADF simplifies orchestration, scheduling, and monitoring of executions.  
This combination of Databricks and ADF allows focusing directly on pipeline development and ELT transformations without the overhead of manually managing containers or clusters.   

## Monitoring  
The use of **Azure Data Factory (ADF)** and **Databricks** provides built-in monitoring and **Data Quality** mechanisms.  
**Azure Data Factory:** In addition to offering detailed logging and metrics, it is possible to set up triggers to send email notifications in case of pipeline failures, for example.  
**Databricks:** With the Unity Catalog enabled, it is possible to use the "Data Monitor" to ensure data quality, along with full data lineage.  
**Other options:** In corporate environments, it is common to use proprietary frameworks and libraries for metric collection and creation, allowing adaptation of monitoring to the organization's needs and integration with tools like Power BI.  

## Cloud Services Configuration  
The integration between **Azure Data Factory (ADF)** and **Azure Databricks** was achieved by creating a Linked Service (ls_dtbks).This approach was chosen for its simplicity. The configuration included:  

**Databricks Workspace URL:** The Databricks workspace URL was provided to establish the connection.  
**Authentication:** Authentication was done using an Access Token generated in Databricks.  
**Cluster Configuration:** The integration was configured to use an existing interactive cluster within Databricks.  

This setup allows ADF to seamlessly connect to Databricks and execute notebooks or jobs in the selected cluster.  
