# Databricks-Notebook-SQL_Automation

<details>

### **Project Title:** End-to-End Streaming Data Ingestion and Analytics Platform on Databricks

***

### **CV/Resume Points**

* Developed and implemented a scalable, multi-layered data architecture (**Bronze, Silver, Gold**) on Databricks to process real-time streaming data from various sources (CSV, JSON, Kafka multiplex).
* Engineered a robust ingestion framework using **Structured Streaming** and **Auto Loader (`cloudFiles`)** to handle incremental data loads, ensuring data idempotency and fault tolerance with checkpointing.
* Designed and built **Change Data Capture (CDC)** logic in the Silver layer to manage Slowly Changing Dimensions (SCD) Type 1 for user profiles, using window functions to handle late and out-of-order data.
* Optimized data pipelines for performance by implementing **`foreachBatch`** for efficient upsert operations, utilizing **`withWatermark`** and **`dropDuplicates`** for stateful stream processing, and leveraging static-stream joins.
* Managed an end-to-end data lifecycle, from raw data ingestion to curated, business-ready aggregations, enabling downstream analytics and reporting, and validated data integrity at each stage using automated assertion tests.

***

### **README.md**

## **End-to-End Data Engineering Project on Databricks**

### **Project Overview**

This project demonstrates the design and implementation of a modern, medallion-style data architecture on the Databricks Lakehouse Platform. The pipeline ingests and processes streaming data from multiple sources, transforming it through a series of stages to create a high-quality, reliable, and analytics-ready dataset. The core objective is to build a robust, scalable, and maintainable data pipeline that can handle real-time data ingestion and complex data transformations, including Change Data Capture (CDC).

The project is structured into three main layers: Bronze, Silver, and Gold, representing raw, validated, and aggregated data, respectively.


### **Data Sources**

The project processes data from several simulated streaming sources, including:

* **`registered_users` (CSV):** New user registrations.
* **`gym_logins` (CSV):** User check-ins and check-outs at gym locations.
* **`kafka_multiplex` (JSON):** A multiplexed Kafka stream containing three distinct topics:
    * **`user_info`:** Change Data Capture (CDC) events for user profile updates.
    * **`workout`:** Start and stop events for user workouts.
    * **`bpm`:** Heart rate data from wearable devices.

### **Pipeline Architecture and Process Flow**

The data flows through a medallion architecture, ensuring data quality and usability at each stage.

#### **1. Bronze Layer (Raw Data Ingestion)**

The Bronze layer is the raw ingestion stage. It reads data directly from the landing zone and stores it in Delta tables without any major transformations. This layer acts as a single source of truth for all raw data.

* **Technology:** Structured Streaming with Auto Loader (`cloudFiles`) is used to incrementally and efficiently ingest new files as they arrive.
* **Process:**
    * **`consume_user_registration`:** Ingests user registration data from CSV files.
    * **`consume_gym_logins`:** Ingests gym login data from CSV files.
    * **`consume_kafka_multiplex`:** Ingests the multiplexed Kafka stream from JSON files. A join with a static `date_lookup` table is performed to enrich the data with time-based attributes.
* **Logic:** The primary logic here is to append all incoming data to the respective Delta tables, preserving the raw state. The `outputMode("append")` is used for this purpose.

#### **2. Silver Layer (Validated & Enriched Data)**

The Silver layer cleanses, refines, and enriches the raw Bronze data. This stage resolves duplicates, applies schemas, and performs necessary joins to create a more reliable and complete dataset for downstream analysis.

* **Technology:** Structured Streaming with `foreachBatch` is used to apply more complex, batch-oriented logic like `MERGE INTO` operations.
* **Process & Logic:**
    * **`upsert_users`:** Deduplicates `registered_users` data based on `user_id` and `device_id` using `withWatermark` and `dropDuplicates`. New records are inserted into the Silver `users` table.
    * **`upsert_gym_logs`:** Inserts new gym logins and updates existing ones with logout timestamps.
    * **`upsert_user_profile`:** Handles Change Data Capture (CDC) for `user_info` records. It uses a **`CDCUpserter`** class to apply window functions to rank records by a timestamp (`updated`), ensuring only the latest update for each user is processed and upserted.
    * **`upsert_user_bins`:** Joins the `users` and `user_profile` tables to create a dimension table with user information, including age bins and location data. This is an example of a stream-to-static join.
    * **`upsert_completed_workouts`:** Joins `workout` start and stop events to create a complete record of workout sessions, using `withWatermark` and a join condition to manage state and handle late-arriving data.
    * **`upsert_workout_bpm`:** Joins the `completed_workouts` and `heart_rate` streams to associate heart rate data with specific workout sessions.

#### **3. Gold Layer (Aggregated & Business-Ready Data)**

The Gold layer contains the final, aggregated data, optimized for business intelligence, reporting, and machine learning. This data is highly curated and ready for consumption.

* **Technology:** Structured Streaming with `foreachBatch` is used to create final aggregate tables.
* **Process & Logic:**
    * **`upsert_workout_bpm_summary`:** Groups heart rate data by `user_id`, `workout_id`, and `session_id` to calculate summary statistics like `min_bpm`, `avg_bpm`, and `max_bpm`. It then joins this with the `user_bins` table to enrich the data with user demographics. This table is optimized for quick analytical queries.

### **Key Skills & Concepts Demonstrated**

* **Delta Lake & Databricks Lakehouse:** Utilizes Delta tables for ACID transactions, schema enforcement, and time travel.
* **Apache Spark Structured Streaming:** Implements a scalable, fault-tolerant streaming pipeline.
* **Data Architecture:** Follows the Bronze-Silver-Gold medallion architecture best practice.
* **Data Ingestion:** Uses Auto Loader for efficient cloud-based file ingestion.
* **Change Data Capture (CDC):** Implements a robust CDC process using Spark's window functions and `MERGE INTO`.
* **Stateful Stream Processing:** Leverages `withWatermark` and joins on streams to handle late and out-of-order data.
* **Data Quality & Validation:** Includes a comprehensive validation framework (`assert_count`, `assert_rows`) to ensure data integrity at each pipeline stage.
* **Performance Optimization:** Employs `foreachBatch` for efficient upserts, broadcasts small DataFrames for optimal joins (`F.broadcast`), and configures Spark properties for better performance.
  
</details>

----

<img width="948" height="515" alt="image" src="https://github.com/user-attachments/assets/40128416-ed2d-4c76-a330-1842870d02e5" />
<img width="942" height="656" alt="image" src="https://github.com/user-attachments/assets/a546a2ae-25f7-49f1-b46e-cf6890ea72a1" />
<img width="918" height="601" alt="image" src="https://github.com/user-attachments/assets/af0e6914-8ddc-4a6b-a48c-2a3a10ec7c97" />

# 10x with ai
## Ask ai to suggest different partition strategy
Step 1: Own the Data Ingestion Pipeline
Step 2: Architect Smart Storage
Step 3: Master Feature Engineering
Step 4: Champion Governance and Ethics
Step 5: Align Pipelines with Business Outcomes

<img width="330" height="334" alt="image" src="https://github.com/user-attachments/assets/84543961-a049-40f6-9bd2-8b4b23379b7b" />
<img width="477" height="466" alt="image" src="https://github.com/user-attachments/assets/d94f9eaf-95d5-47a3-9f0d-69e0be12a0f5" />
<img width="329" height="245" alt="image" src="https://github.com/user-attachments/assets/96f60450-4376-4c43-ba46-d92ae67633bb" />
<img width="354" height="276" alt="image" src="https://github.com/user-attachments/assets/4f54fa20-c3dc-4174-ae66-6c4f913359da" />
<img width="409" height="341" alt="image" src="https://github.com/user-attachments/assets/40272478-5053-4c1f-9362-6df6ec3d70a8" />
<img width="469" height="286" alt="image" src="https://github.com/user-attachments/assets/c69165ad-470e-4dfd-974c-454365018620" />




<img width="329" height="264" alt="image" src="https://github.com/user-attachments/assets/8b138c31-7053-4527-9fd6-33c9662b35a0" />
<img width="295" height="264" alt="image" src="https://github.com/user-attachments/assets/f53948fe-de9e-4c40-939a-07f8f07701de" />
