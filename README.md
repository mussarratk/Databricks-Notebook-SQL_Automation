
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
- Lambda function role - permissions to go into S3, write to DynamoDB, and send logs to CloudWatch.

  
<img width="1354" height="521" alt="image" src="https://github.com/user-attachments/assets/363fee42-a0a1-4316-ae97-7c81baa9b5a0" />
<img width="1347" height="502" alt="image" src="https://github.com/user-attachments/assets/cd7602af-269c-4205-b5b2-3ff12f9bdaf2" />



Step 4 - The Lambda Logic (FinSight-File-Validator)You will write this in Python 3.x using the boto3 library.Key Logic Highlights:Parsing the Filename: Use .split('_') to extract the region (e.g., transactions_Asia_20230101.csv).Checking Content: Use body.decode('utf-8').splitlines() to check if the length of lines is $> 1$.Moving Files: In S3, there is no "move" command. You must Copy the object to the new path (validated/dt=.../) and then Delete the original from incoming/.

- ETL - AWS lambda manual trigger - test
- ETL - lambda automated trigger
- ETL - automation using s3 event 

## **Event-Driven ETL (Extract, Transform, Load) Pipeline**. 

## How S3 Event Automation Works in this Project
Instead of a person or a schedule checking the `incoming/` folder, the S3 bucket itself acts as a "sensor".

### 1. The Trigger (The "Event")
*   **Action**: A source system (like the Asia or EMEA region) uploads `transactions_Asia_20260504.csv` to your S3 bucket.
*   **Event**: S3 generates an `s3:ObjectCreated:*` JSON notification.

### 2. The Orchestrator (Lambda)
*   AWS sees the event and immediately wakes up your **FinSight-File-Validator** Lambda function.
*   The event details (bucket name and file path) are passed into the `event` parameter of your Python code.

### 3. The "Light" ETL Process
While Task 1 is technically a **Validation Layer**, it performs the core functions of ETL:
*   **Extract**: Lambda reads the metadata and the first few lines of the CSV from S3.
*   **Transform (Validation)**: It checks the file format and ensures it isn't just a header. 
*   **Load (Routing)**: It "loads" the file into a structured destination (`validated/dt=YYYY-MM-DD/`) and logs the entry into the DynamoDB audit table.

---

## Why this is better for FinSight
The client mentioned they had issues with **partial uploads** and **duplicate processing**. Automation via S3 events solves this because:

*   **Atomicity**: The event only triggers *after* the file is fully uploaded, preventing the "partial upload" errors the analytics team complained about.
*   **Decoupling**: The regions sending the data don't need to know how the data is processed. They just "drop" the file and walk away.
*   **Instant Visibility**: Because your Lambda logs to DynamoDB immediately, the "missing reports" issue is solved; the team can see exactly when a file arrived and if it passed the gatekeeper.


<img width="1363" height="563" alt="image" src="https://github.com/user-attachments/assets/433a2e4e-5d58-455b-b8b1-0350481d77e5" />
<img width="1361" height="547" alt="image" src="https://github.com/user-attachments/assets/dc0adba0-afb4-481f-9b17-2643bb9501dd" />
<img width="1354" height="527" alt="image" src="https://github.com/user-attachments/assets/66967cf6-f85b-480f-be17-e8e270f05be6" />
<img width="1361" height="564" alt="image" src="https://github.com/user-attachments/assets/00978b5f-450f-4840-ac34-6d61f5fd4709" />
<img width="1358" height="540" alt="image" src="https://github.com/user-attachments/assets/1aef9c3a-9576-43cc-932a-d31cd02f533b" />


<details>

To handle a dynamic date partition like `dt=YYYY-MM-DD` that changes every day, you must avoid hardcoding the date in your Lambda function. Instead, your code should **dynamically extract the date** from the S3 path (`full_key`) that triggered the event.

Since your files are organized as `incoming/dt=YYYY-MM-DD/filename.csv`, the Lambda can "read" the folder name and reuse it for the destination path.

### 1. Updated Dynamic Lambda Logic
Update your `FinSight-File-Validator` code to use this logic. It splits the S3 path to find the date string automatically.

```python
import boto3
import uuid
from datetime import datetime

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('finsight-ingestion-audit')

def lambda_handler(event, context):
    # 1. Capture the full path (key) from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    full_key = event['Records'][0]['s3']['object']['key'] 
    
    # Example full_key: incoming/dt=2025-09-01/transactions_asia_2025-09-01.csv
    parts = full_key.split('/')
    
    # 2. Extract the date partition and filename
    # parts[0] = 'incoming', parts[1] = 'dt=2025-09-01', parts[2] = 'filename'
    date_partition = parts[1] if len(parts) > 2 else "dt=unknown"
    filename = parts[-1]
    
    # 3. Extract Region
    try:
        region = filename.split('_')[1].upper()
    except:
        region = "UNKNOWN"

    # --- VALIDATION LOGIC ---
    status = "Validated"
    reason = ""
    
    if not filename.lower().endswith('.csv'):
        status = "Failed"
        reason = "Invalid File Format"
    else:
        # Read file to check content
        response = s3.get_object(Bucket=bucket, Key=full_key)
        lines = response['Body'].read().decode('utf-8').splitlines()
        if len(lines) <= 1:
            status = "Failed"
            reason = "Empty or Header-only file"

    # 4. DYNAMIC MOVEMENT
    # We reuse the date_partition extracted from the 'incoming' path
    target_root = "validated" if status == "Validated" else "failed"
    new_key = f"{target_root}/{date_partition}/{filename}"
    
    # Move the file
    s3.copy_object(Bucket=bucket, CopySource={'Bucket': bucket, 'Key': full_key}, Key=new_key)
    s3.delete_object(Bucket=bucket, Key=full_key)

    # 5. AUDIT LOGGING
    table.put_item(Item={
        'file_name': filename,
        'serial_no': str(uuid.uuid4()),
        'region': region,
        'status': status,
        'failed_reason': reason,
        'file_path': f"s3://{bucket}/{new_key}",
        'processed_timestamp': datetime.now().isoformat()
    })
```

---

### 2. Why this solves the "Changing Date" problem:
*   **Automatic Extraction**: By using `full_key.split('/')`, the code grabs whatever folder name exists between `incoming/` and the filename. If tomorrow the folder is `dt=2025-09-02`, the code will automatically pick that up.
*   **Trigger Stability**: Your S3 trigger should still be set to Prefix: `incoming/`. Because S3 is a flat-file system, a trigger on `incoming/` will fire for *any* file added to *any* subfolder inside it.

### 3. Handling the 4 files per day
When you or the source system uploads the 4 files (Asia, EMEA, NA, LATAM) into the new date folder:
1.  S3 will fire **4 separate events**.
2.  Your Lambda will run **4 separate times** (concurrently).
3.  Each run will look at its specific `full_key`, validate the file, move it to the matching `dt=` folder in `validated/` or `failed/`, and create a unique entry in DynamoDB.

### 4. Verification Check
After updating your code and uploading the files from your screenshot (**image_98bd57.png**):
*   **DynamoDB**: Should show 4 entries.
*   **S3**: `incoming/dt=2025-09-01/` should be empty.
*   **S3**: `validated/dt=2025-09-01/` should have 3 files (Asia, EMEA, NA).
*   **S3**: `failed/dt=2025-09-01/` should have 1 file (LATAM.pdf).

 
</details>
- To handle a dynamic date partition like dt=YYYY-MM-DD that changes every day, you must avoid hardcoding the date in your Lambda function. Instead, your code should dynamically extract the date from the S3 path (full_key) that triggered the event.


This is a classic "Data Engineering 101" challenge: building a **Landing Zone** with a **Gatekeeper**. By implementing this, you solve the "garbage in, garbage out" problem before the data ever touches your analytics engine.

Here is your step-by-step guide to building the FinSight Ingestion & Validation layer.

---

## 1. Architectural Overview
The workflow follows a serverless event-driven pattern. When a file hits S3, it triggers a Lambda function that acts as the "brain," deciding where the file goes and logging the action in DynamoDB.



---

## 2. Infrastructure Setup (AWS Console)

### Step A: S3 Bucket
1. Create a bucket: `finsight-de-[yourname]`.
2. Create three top-level folders: `incoming/`, `validated/`, and `failed/`.

### Step B: DynamoDB Table
1. Table Name: `finsight-ingestion-audit`.
2. **Partition Key**: `file_name` (String).
3. (Optional but recommended) **Sort Key**: `processed_timestamp` (String).

### Step C: IAM Policy (The "Permissions")
Your Lambda needs a Role with a policy that allows:
*   **S3**: `GetObject`, `PutObject`, `DeleteObject` (to move files).
*   **DynamoDB**: `PutItem`.
*   **CloudWatch**: `CreateLogGroup`, `PutLogEvents` (for debugging).

---

## 3. The Lambda Logic (`FinSight-File-Validator`)

You will write this in **Python 3.x** using the `boto3` library. 

### Key Logic Highlights:
*   **Parsing the Filename**: Use `.split('_')` to extract the region (e.g., `transactions_Asia_20230101.csv`).
*   **Checking Content**: Use `body.decode('utf-8').splitlines()` to check if the length of lines is $> 1$.
*   **Moving Files**: In S3, there is no "move" command. You must **Copy** the object to the new path (`validated/dt=.../`) and then **Delete** the original from `incoming/`.

### Implementation Outline:
```python
import boto3
import uuid
from datetime import datetime

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('finsight-ingestion-audit')

def lambda_handler(event, context):
    # 1. Get Bucket and File Name from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key'] # e.g., incoming/file.csv
    filename = key.split('/')[-1]
    
    # 2. Extract Region (Assuming pattern: transactions_REGION_date.csv)
    try:
        region = filename.split('_')[1]
    except IndexError:
        region = "Unknown"

    # 3. Perform Validations
    status = "Validated"
    reason = ""
    
    # Validation 1: Format
    if not filename.endswith('.csv'):
        status = "Failed"
        reason = "Invalid File Format"
    
    # Validation 2: Content (Read first 2 lines)
    else:
        response = s3.get_object(Bucket=bucket, Key=key)
        lines = response['Body'].read().decode('utf-8').splitlines()
        if len(lines) <= 1:
            status = "Failed"
            reason = "Empty or Header-only file"

    # 4. Determine Destination & Move File
    dt_partition = datetime.now().strftime("%Y-%m-%d")
    target_folder = "validated" if status == "Validated" else "failed"
    new_key = f"{target_folder}/dt={dt_partition}/{filename}"
    
    s3.copy_object(Bucket=bucket, CopySource={'Bucket': bucket, 'Key': key}, Key=new_key)
    s3.delete_object(Bucket=bucket, Key=key)

    # 5. Log to DynamoDB
    table.put_item(Item={
        'file_name': filename,
        'serial_no': str(uuid.uuid4()),
        'region': region,
        'status': status,
        'failed_reason': reason,
        'file_path': f"s3://{bucket}/{new_key}",
        'processed_timestamp': datetime.now().isoformat()
    })
```

---

## 4. Setting the Trigger
Once the code is deployed:
1. Go to your Lambda **Configuration** tab.
2. Select **Triggers** -> **Add Trigger**.
3. Choose **S3**.
4. Select your bucket.
5. **Event type**: `All object create events`.
6. **Prefix**: `incoming/` (This ensures the Lambda only runs when files land in the "incoming" folder, preventing an infinite loop).

---

## 5. Testing Your Solution
To verify your work, upload these three files to `incoming/`:

| File Name | Content | Expected Result |
| :--- | :--- | :--- |
| `transactions_Asia_01.csv` | Header + 5 rows | Moves to `validated/`, Logged as Success |
| `transactions_NA_02.txt` | Any text | Moves to `failed/`, Reason: Invalid Format |
| `transactions_LATAM_03.csv` | Header only | Moves to `failed/`, Reason: Empty/Header-only |

### Why this fixes the Client's issues:
*   **No more duplicates:** Once a file is processed, it is deleted from `incoming/`.
*   **High Visibility:** The DynamoDB table acts as a real-time dashboard for the analytics team.
*   **Organization:** Date partitioning (`dt=YYYY-MM-DD`) makes it much easier for AWS Glue or Athena to crawl the data later.


- Load files to s3

<img width="1360" height="539" alt="image" src="https://github.com/user-attachments/assets/50e731a8-467c-49f2-9ddf-83e021e99da6" />
<img width="815" height="524" alt="image" src="https://github.com/user-attachments/assets/cdba9558-3b77-41b9-aa20-acf3884432c4" />

- Cloudwatch
<img width="1365" height="533" alt="image" src="https://github.com/user-attachments/assets/e53dd1bb-bd5f-4878-b546-841236a93d6e" />

<img width="1360" height="491" alt="image" src="https://github.com/user-attachments/assets/19883716-7424-402c-8465-d6298e8dc676" />
<img width="1363" height="524" alt="image" src="https://github.com/user-attachments/assets/fadd1603-32c7-4086-b2ff-c56e0f7f19e8" />


- Dynamodb
<img width="1360" height="525" alt="image" src="https://github.com/user-attachments/assets/0d695570-52d2-40ce-8437-3eb816893b0a" />
<img width="1365" height="557" alt="image" src="https://github.com/user-attachments/assets/5ec4feec-f221-4e56-949c-dfc5d6df7064" />



---
- 4 files in failed/ folder

 
</details>
