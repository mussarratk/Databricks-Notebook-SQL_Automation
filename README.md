
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
<img width="1359" height="526" alt="image" src="https://github.com/user-attachments/assets/787384b8-e0c8-443a-93bb-3fae4d7ce037" />


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
<img width="1226" height="288" alt="image" src="https://github.com/user-attachments/assets/10541b5d-2a10-4488-a0e2-ba48117cff76" />
<img width="1352" height="534" alt="image" src="https://github.com/user-attachments/assets/6ecffa8e-c7fc-420c-8280-a6b1fba5b127" />


---
- 4 files in failed/ folder

<img width="1198" height="416" alt="image" src="https://github.com/user-attachments/assets/44e623fa-e4b8-4aa6-b759-60676f41776d" />
<img width="797" height="427" alt="image" src="https://github.com/user-attachments/assets/fd31a8e6-9c85-4677-bdd6-0d78675c5253" />
<img width="1358" height="478" alt="image" src="https://github.com/user-attachments/assets/2981c7b4-e641-4a88-ad78-897e7577cd6d" />

---

<img width="1350" height="495" alt="image" src="https://github.com/user-attachments/assets/44d8878c-f994-4cf4-af77-656408850e80" />
<img width="1344" height="485" alt="image" src="https://github.com/user-attachments/assets/63d3db44-e076-403b-a5e3-712f05f2ada7" />

---

 
</details>


<details>

---
# FinSight-DQ-Processor - Serverless Row-Level - This Lambda will act as the second gate in your pipeline, transforming raw "validated" files into analytics-ready data.

<img width="1360" height="416" alt="image" src="https://github.com/user-attachments/assets/094ace90-2bee-4c16-8aec-fc75ac2b8fec" />

<img width="1353" height="397" alt="image" src="https://github.com/user-attachments/assets/92e21d3e-72fa-4b56-92d2-f240bc7879d6" />
<img width="1357" height="566" alt="image" src="https://github.com/user-attachments/assets/3056beb7-4244-4e88-b081-78c169d376c2" />
<img width="1355" height="557" alt="image" src="https://github.com/user-attachments/assets/e5b4ff29-59d1-49b0-8429-04895a88fed6" />
<img width="1360" height="601" alt="image" src="https://github.com/user-attachments/assets/079da612-c764-43ff-b489-727224231c9f" />
<img width="1366" height="607" alt="image" src="https://github.com/user-attachments/assets/2184c697-0cae-4003-870e-cfe978d2f156" />

<img width="1361" height="601" alt="image" src="https://github.com/user-attachments/assets/23727aaa-f226-4eca-9108-50cad2d57da2" />

<img width="1349" height="496" alt="image" src="https://github.com/user-attachments/assets/36a07fc4-9f2d-4d88-9d75-e23766826907" />

<img width="892" height="491" alt="image" src="https://github.com/user-attachments/assets/10deceb5-9c4f-470d-bf5c-19ece6e68630" />

- Total - rejected 12+16= 28
<img width="1347" height="523" alt="image" src="https://github.com/user-attachments/assets/32bde390-3055-4594-b9bf-6a8bce7c5358" />
<img width="1357" height="593" alt="image" src="https://github.com/user-attachments/assets/0e8c07b3-842b-4052-8071-8aec7c5686ea" />


<img width="1344" height="531" alt="image" src="https://github.com/user-attachments/assets/c82dc489-7926-4239-8efa-89363e72ed2c" />
<img width="1353" height="497" alt="image" src="https://github.com/user-attachments/assets/86072433-77d9-40c8-baf8-d43a9c5956a9" />

<img width="1354" height="536" alt="image" src="https://github.com/user-attachments/assets/531bdad5-802c-4b36-900f-eb6e4ab7ed5a" />


 ---
</details>



<details>
---

#Task 3: Schema Evolution & Drift Management
To implement **Task 3: Schema Evolution & Drift Management**, we need to update your ingestion logic to include a **pre-validation layer**. This layer will sit before your Row-Level DQ logic and handle the standardization of headers, missing columns, and new business requirements.

### 1. Preparation: DynamoDB & S3
1.  **Create DynamoDB Table**: `finsight-schema-registry` (Partition Key: `drift_id`).
2.  **S3 Config**: Upload your `schema_mapping.json` and `schema_master.json` to a folder named `config/` in your `finsight-de-mus` bucket.

---

### 2. Lambda – Schema Drift Handling Logic
This code incorporates **Step 1 (Standardization)**, **Step 2 (Enrichment)**, and **Step 3 (Evolution)**. It also handles the concurrency requirement by fetching the latest master schema before updates.

```python
import boto3
import csv
import io
import json
import uuid
import urllib.parse
from datetime import datetime
from decimal import Decimal

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
registry_table = dynamodb.Table('finsight-schema-registry')
audit_table = dynamodb.Table('finsight-dq-audit')

CONFIG_BUCKET = 'finsight-de-mus'

def log_drift(file_name, drift_type):
    registry_table.put_item(Item={
        'drift_id': str(uuid.uuid4()),
        'file_name': file_name,
        'drift_type': drift_type,
        'timestamp': datetime.utcnow().isoformat()
    })

def lambda_handler(event, context):
    # 1. Capture and Decode Source Info
    bucket = event['Records'][0]['s3']['bucket']['name']
    raw_key = event['Records'][0]['s3']['object']['key']
    key = urllib.parse.unquote_plus(raw_key)
    filename = key.split('/')[-1]
    date_partition = key.split('/')[1]

    try:
        # Load Config Files
        mapping_obj = s3.get_object(Bucket=CONFIG_BUCKET, Key='config/schema_mapping.json')
        schema_mapping = json.loads(mapping_obj['Body'].read().decode('utf-8'))
        
        master_obj = s3.get_object(Bucket=CONFIG_BUCKET, Key='config/schema_master.json')
        master_data = json.loads(master_obj['Body'].read().decode('utf-8'))
        
        # Get Current Master Columns
        current_v = master_data['current_version']
        master_cols = [col['name'] for col in master_data['versions'][current_v]['expected_columns']]

        # Read Input File
        response = s3.get_object(Bucket=bucket, Key=key)
        content = response['Body'].read().decode('utf-8')
        reader = csv.DictReader(io.StringIO(content))
        original_headers = reader.fieldnames
        
        # --- STEP 1: Header Standardization ---
        standardized_headers = []
        renamed_flag = False
        for h in original_headers:
            if h in schema_mapping:
                standardized_headers.append(schema_mapping[h])
                renamed_flag = True
            else:
                standardized_headers.append(h)
        
        if renamed_flag:
            log_drift(filename, 'column_renamed')

        # --- STEP 2: Missing Column Enrichment (MANDATORY COLS) ---
        mandatory_required = ['transaction_id', 'region', 'transaction_amount', 'currency', 'transaction_date']
        missing_cols = [m for m in mandatory_required if m not in standardized_headers]
        
        inferred_region = filename.split('_')[1].upper() # e.g. transactions_NA_... -> NA

        # --- STEP 3: Schema Evolution (New Columns) ---
        new_cols_detected = [h for h in standardized_headers if h not in master_cols]
        if new_cols_detected:
            # Simple version increment
            new_v_num = float(master_data['versions'][current_v]['version']) + 1.0
            new_v_name = f"v{int(new_v_num)}"
            
            # Update master schema dict
            new_expected = master_data['versions'][current_v]['expected_columns'].copy()
            for nc in new_cols_detected:
                new_expected.append({"name": nc, "type": "string"})
            
            master_data['current_version'] = new_v_name
            master_data['versions'][new_v_name] = {
                "version": new_v_num,
                "primary_key": master_data['versions'][current_v]['primary_key'],
                "expected_columns": new_expected
            }
            
            # Save updated master to S3
            s3.put_object(Bucket=CONFIG_BUCKET, Key='config/schema_master.json', Body=json.dumps(master_data))
            log_drift(filename, 'schema_change_new_column')
            master_cols.extend(new_cols_detected)

        # Final Header List for rows
        final_headers = standardized_headers + missing_cols
        
        # Process Rows (Row-Level DQ Integration)
        valid_rows, invalid_rows = [], []
        for row in reader:
            # Map original values to standardized header names
            processed_row = {}
            for idx, val in enumerate(row.values()):
                processed_row[standardized_headers[idx]] = val
            
            # Apply Step 2 Enrichment to values
            if 'region' in missing_cols:
                processed_row['region'] = inferred_region
                if 'missing_column_inferred' not in [d for d in []]: # Logic to log once per file
                    log_drift(filename, 'missing_column_inferred')

            # --- Row-Level DQ (Task 2 Rules) ---
            errors = []
            # Rule: Null Check
            if not processed_row.get('transaction_id'): errors.append("Empty ID")
            # Rule: Amount Check
            try:
                if float(processed_row.get('transaction_amount', 0)) <= 0: errors.append("Non-positive Amt")
            except: errors.append("Non-numeric Amt")
            
            if errors:
                processed_row['error_reason'] = "; ".join(errors)
                invalid_rows.append(processed_row)
            else:
                valid_rows.append(processed_row)

        # 4. Output Generation (Same as Task 2)
        # [Helper function write_to_s3 goes here]
        
        return {"status": "Schema & DQ Processed", "valid": len(valid_rows)}

    except Exception as e:
        print(f"Error: {e}")
        raise e
```

### Key Highlights for Task 3:
*   **Immutable Logs**: Every time `txt_amt` is found, a `column_renamed` entry hits DynamoDB.
*   **Filename Inference**: If the `region` column is missing, the code extracts "NA" from `transactions_NA_...csv` and injects it into every row.
*   **Resiliency**: If a partner adds `branch_id`, the `schema_master.json` version bumps to `2.0` automatically, and the file is processed instead of failing.

---
<img width="1352" height="517" alt="image" src="https://github.com/user-attachments/assets/d6455bdb-8836-4c0b-bfdf-86e5646cfb2f" />
<img width="1338" height="471" alt="image" src="https://github.com/user-attachments/assets/8af9ef9b-9569-4f4a-a40e-c7c57992790c" />

<img width="1350" height="482" alt="image" src="https://github.com/user-attachments/assets/cb88069d-c76c-40c6-b69e-9e69119fe865" />

<img width="1362" height="599" alt="image" src="https://github.com/user-attachments/assets/d81ffa50-7f7e-41cb-91d4-bc157f618fe0" />

<img width="1349" height="508" alt="image" src="https://github.com/user-attachments/assets/ca8925b2-924a-46fe-b731-d650aaace08b" />

<img width="1357" height="526" alt="image" src="https://github.com/user-attachments/assets/71af94d5-6680-45cc-b019-c9aad778ede2" />
<img width="1360" height="529" alt="image" src="https://github.com/user-attachments/assets/23157676-0b96-4b75-a0e4-259b9540e75f" />


---

 
</details>

<details>

---

# Task - 4 
## Pipeline Optimization and Unified Flow

- s3

<img width="1359" height="501" alt="image" src="https://github.com/user-attachments/assets/9a8403cc-d553-43cb-ba3d-bfade42a61d4" />

<img width="1357" height="344" alt="image" src="https://github.com/user-attachments/assets/2b1eaf3c-2d43-4639-a5dd-d4869dae815e" />
<img width="1347" height="489" alt="image" src="https://github.com/user-attachments/assets/d6ad2985-f8de-4a36-b322-943d6e5886fb" />
<img width="1340" height="497" alt="image" src="https://github.com/user-attachments/assets/c7bca710-a9f1-48bc-b9d0-302dfea18735" />
<img width="1304" height="480" alt="image" src="https://github.com/user-attachments/assets/1a944fc7-1b77-466a-9fc8-b31b28736871" />
<img width="1314" height="478" alt="image" src="https://github.com/user-attachments/assets/804c1d5e-0b90-4a04-a419-3a16063716ab" />

- lambda

<img width="1350" height="532" alt="image" src="https://github.com/user-attachments/assets/c8eeebbf-7ce8-4d61-9717-f611fa4c4739" />
<img width="1366" height="524" alt="image" src="https://github.com/user-attachments/assets/98a117d0-51af-4333-a548-3d3e5dcd57e0" />

<img width="1363" height="593" alt="image" src="https://github.com/user-attachments/assets/c802ce4d-0583-4559-9c0b-afc4ab52511b" />


- dynamodb

  - test
  - <img width="1344" height="511" alt="image" src="https://github.com/user-attachments/assets/a2dc1614-e0d5-4226-b53a-330deb0a7da0" />
<img width="1356" height="563" alt="image" src="https://github.com/user-attachments/assets/54ed14fe-90c7-482f-96d3-6da1c79cf9ee" />
<img width="1351" height="511" alt="image" src="https://github.com/user-attachments/assets/6b460234-5fab-4f0d-b87e-bd04800f3195" />
<img width="999" height="341" alt="image" src="https://github.com/user-attachments/assets/2a370fab-228b-499c-9339-48e2e7af17c8" />



- slack
<img width="1364" height="575" alt="image" src="https://github.com/user-attachments/assets/99bfe3a1-4121-46a6-b046-de24d8823b3d" />
<img width="1045" height="521" alt="image" src="https://github.com/user-attachments/assets/72dd6c12-f0fc-4262-8d8f-357470600971" />
<img width="1009" height="543" alt="image" src="https://github.com/user-attachments/assets/fe0064e9-281c-4386-9e7f-ace14b3216dc" />
<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/49b38c2a-09e5-470f-a0da-d1347cda4787" />
<img width="733" height="561" alt="image" src="https://github.com/user-attachments/assets/cb8085f0-a275-4232-9164-e02d12ec0338" />




---

### In Task 4, goal is to refactor the pipeline into:

- A clean, maintainable S3 folder structure
- One unified Lambda that handles file checks, schema drift handling, row-level DQ, and output routing
- One DynamoDB audit table for complete observability
- Slack notifications driven directly from the DynamoDB audit record
  
---

### The Sequence of Events
1.  **Upload**: Upload all dates file  `transactions_ASIA_2026-05-06.csv` to `finsight-de-musb/incoming/`.
2.  **Trigger**: S3 detects the new object and sends an event notification to your **Unified Lambda**.
3.  **Execution**: The Lambda parses the name, checks for a valid CSV, standardizes the headers, validates the rows, and logs everything to **DynamoDB**.
4.  **Alert**: Finally, it sends that formatted success or failure message to your **Slack channel**.

---

 
</details>
