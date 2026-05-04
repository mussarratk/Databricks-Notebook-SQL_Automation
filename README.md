
-----

# FinSight Data Engineering Pipeline

### *Serverless Global Financial Data Ingestion, Schema Governance, and Automated Quality Assurance*

[](https://aws.amazon.com/)
[](https://www.python.org/)
[](https://www.serverless.com/)

## 📌 Project Overview

**FinSight Corp** handles high-velocity financial transactions across four major global regions (Asia, EMEA, NA, LATAM). This project implements an end-to-end **Data Engineering Pipeline** that automates the transition from raw, unpredictable CSV uploads to high-fidelity, analytics-ready datasets.

The pipeline solves critical data engineering challenges:

  * **Data Integrity:** Prevents corrupted or malformed files from entering the lake.
  * **Schema Evolution:** Dynamically handles upstream column changes without pipeline breakage.
  * **Business Logic Enforcement:** Validates region-specific currency codes and transaction values.
  * **Observability:** Provides real-time Slack alerts and a DynamoDB-backed audit trail.

-----

## 🏗 Architecture & Workflow

The architecture is 100% serverless, ensuring high availability and cost-efficiency.

1.  **Ingestion:** Files land in `S3: /incoming`.
2.  **Trigger:** S3 Event Notifications invoke an **AWS Lambda** function.
3.  **Validation (Two-Tier):**
      * **File-Level:** Checks naming conventions, region validity, and file corruption.
      * **Row-Level:** Evaluates every record against schema definitions and business rules (e.g., Currency vs. Region mapping).
4.  **Schema Governance:** Uses `schema_mapping.json` for normalization and `schema_master.json` for versioning and drift detection.
5.  **Storage & Partitioning:**
      * **Clean Data:** Moved to `processed/` partitioned by date (`dt=YYYY-MM-DD`).
      * **Quarantine:** Invalid files and rejected rows (with error metadata) are isolated for debugging.
6.  **Observability:** Updates **DynamoDB** with a Data Quality (DQ) score and dispatches alerts via **Slack Webhooks**.

-----

## 🛠 Tech Stack

| Component | Service | Role |
| :--- | :--- | :--- |
| **Cloud Provider** | AWS | Primary Infrastructure |
| **Compute** | AWS Lambda (Python) | Unified Processing Engine |
| **Storage (Lake)** | Amazon S3 | Bronze & Silver Data Zones |
| **NoSQL Database** | Amazon DynamoDB | Audit Logging & Schema Metadata |
| **Monitoring** | Slack API | Real-time Operational Alerts |
| **Security** | AWS IAM | Least-Privilege Role Access |

-----

## 📂 Data Quality & Business Rules

The pipeline enforces strict cross-field validation to ensure financial accuracy:

| Rule Category | Description | Failure Action |
| :--- | :--- | :--- |
| **Structural** | File must follow `transactions_<region>_<date>.csv` | File Quarantined |
| **Mandatory** | `transaction_id` cannot be NULL | Row Rejected |
| **Numeric** | `transaction_amount` must be \> 0 and numeric | Row Rejected |
| **Geospatial** | Currency must match Region (e.g., `Asia` -\> `INR`) | Row Rejected |

### Data Quality Score Calculation

To provide instant visibility into data health, the pipeline calculates a DQ score for every batch:
$$\text{DQ Score} = \left( \frac{\text{Valid Records}}{\text{Total Records}} \right) \times 100$$

-----

## 🔧 Configuration & Schema Evolution

The pipeline is configuration-driven, allowing for non-breaking changes:

  * **`schema_mapping.json`**: Acts as a translation layer (e.g., mapping `txn_id` or `curr` to standard internal naming).
  * **`schema_master.json`**: Defines the "Source of Truth." If a new valid column is detected, the pipeline logs a **Schema Drift Event** and updates the master version automatically.

-----

## 📁 Repository Structure

```text
finsight-de-pipeline/
├── lambda_function.py      # Core processing logic
├── utils/
│   ├── validator.py        # Row-level business rules
│   └── slack_helper.py     # Notification logic
├── config/                 # Governance files
│   ├── schema_mapping.json 
│   └── schema_master.json
├── iam_policy.json         # Security configuration
└── README.md
```

-----

## 🚀 Getting Started

### Prerequisites

  * AWS Account with S3, Lambda, and DynamoDB access.
  * Python 3.9+
  * A Slack Webhook URL for notifications.

### Setup

1.  **S3 Buckets:** Create your bucket and subfolders (`incoming/`, `processed/`, `quarantine/`, `config/`).
2.  **DynamoDB:** Create a table named `finsight-unified-audit` with `file_name` as the Partition Key.
3.  **Lambda Deployment:**
      * Deploy the `lambda_function.py`.
      * Set Environment Variables: `SLACK_WEBHOOK_URL`, `DYNAMODB_TABLE`.
      * Configure S3 Trigger on the `incoming/` prefix.
4.  **Upload Config:** Place your JSON config files in the `config/` folder.

-----

## 📈 Results & Impact

  * **Zero Downtime Schema Changes:** Handled column renames dynamically without updating code.
  * **100% Traceability:** Every failed row is preserved with a specific `error_reason` for the business team to review.
  * **Proactive Monitoring:** Reduced "Mean Time to Discovery" for data issues from hours to seconds via Slack alerts.

-----

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.


---

<details>

---

Step A: S3 Bucket - setup
<img width="1359" height="454" alt="image" src="https://github.com/user-attachments/assets/7c6cf714-995b-4915-a14c-78d902e5ac0c" />

Step B: DynamoDB Table

<img width="1358" height="552" alt="image" src="https://github.com/user-attachments/assets/e0b955b9-29aa-46f3-a3e5-c6fe397ab67b" />

Step C: IAM Policy (The "Permissions")

- Lambda needs a Role with a policy that allows:
- S3: GetObject, PutObject, DeleteObject (to move files).
- DynamoDB: PutItem.
- CloudWatch: CreateLogGroup, PutLogEvents (for debugging).

<img width="1354" height="521" alt="image" src="https://github.com/user-attachments/assets/363fee42-a0a1-4316-ae97-7c81baa9b5a0" />



Step 4 - The Lambda Logic (FinSight-File-Validator)You will write this in Python 3.x using the boto3 library.Key Logic Highlights:Parsing the Filename: Use .split('_') to extract the region (e.g., transactions_Asia_20230101.csv).Checking Content: Use body.decode('utf-8').splitlines() to check if the length of lines is $> 1$.Moving Files: In S3, there is no "move" command. You must Copy the object to the new path (validated/dt=.../) and then Delete the original from incoming/.










---


 
</details>
