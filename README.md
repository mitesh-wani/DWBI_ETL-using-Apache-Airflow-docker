![WhatsApp Image 2026-04-08 at 1 03 25 AM](https://github.com/user-attachments/assets/26615bca-6ece-4390-b65b-ad0396dc5c76)
# 🚀 Airflow ETL Pipeline with PostgreSQL & Google Sheets

## 📌 Project Overview

This project implements a **production-style ETL (Extract, Transform, Load) data pipeline** using:

* **Apache Airflow** → Workflow orchestration
* **PostgreSQL** → Metadata database (Airflow backend)
* **Docker** → Containerized environment
* **Pandas** → Data processing
* **Google Sheets API** → Data output layer

The pipeline reads raw delivery data from a CSV file, performs advanced transformations, and loads the cleaned dataset into Google Sheets.

---

## 🧠 Architecture

```
                +----------------------+
                |   Raw CSV Dataset    |
                | (delivery_data.csv)  |
                +----------+-----------+
                           |
                           v
                +----------------------+
                |     Airflow DAG      |
                |  Extract → Transform |
                |        → Load        |
                +----------+-----------+
                           |
        +------------------+------------------+
        |                                     |
        v                                     v
+----------------------+         +--------------------------+
|  PostgreSQL (MetaDB) |         |   Google Sheets Output   |
| (Airflow backend DB) |         |   (Final Clean Dataset)  |
+----------------------+         +--------------------------+
```

---

## 🎯 Objectives

* Build a **modular ETL pipeline**
* Handle **large datasets efficiently using chunking**
* Implement **data cleaning, transformation, and feature engineering**
* Integrate **Airflow with external services (Google Sheets)**
* Use **Docker for reproducibility**

---

## 📂 Project Structure

```
airflow-etl-project/
│
├── dags/
│   ├── etl_pipeline.py
│   └── credentials.json
│
├── data/
│   └── delivery_data_with_restaurants.csv
│
├── logs/
├── plugins/
├── config/
│
└── docker-compose.yaml
```

---

## ⚙️ Technologies Used

| Component     | Technology        |
| ------------- | ----------------- |
| Orchestration | Apache Airflow    |
| Database      | PostgreSQL        |
| Processing    | Pandas            |
| Container     | Docker            |
| API           | Google Sheets API |

---

## 🐳 Docker Setup

### docker-compose.yaml

```yaml
version: '3'

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    ports:
      - "5432:5432"

  airflow:
    image: apache/airflow:2.8.1
    depends_on:
      - postgres
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
      - AIRFLOW__CORE__LOAD_EXAMPLES=False
    volumes:
      - ./dags:/opt/airflow/dags
      - ./data:/opt/airflow/dags/data
    ports:
      - "8080:8080"
    command: >
      bash -c "pip install pandas gspread oauth2client &&
               airflow db init &&
               airflow users create --username admin --password admin --firstname admin --lastname user --role Admin --email admin@example.com &&
               airflow scheduler & airflow webserver"
```

---

## 💻 Installation & Execution Steps

### 1. Clone or Create Project

```bash
mkdir airflow-etl-project
cd airflow-etl-project
mkdir dags data logs plugins config
```

---

### 2. Start Docker Containers

```bash
docker-compose up -d
```

---

### 3. Verify Containers

```bash
docker ps
```

---

### 4. Access Airflow UI

```
http://localhost:8080
```

Login:

```
Username: admin
Password: admin
```

---

## 🔐 Google Sheets Setup

### Step 1: Create Service Account

1. Go to Google Cloud Console
2. Enable:

   * Google Sheets API
   * Google Drive API
3. Create Service Account
4. Download `credentials.json`

---

### Step 2: Place Credentials

```
dags/credentials.json
```

---

### Step 3: Share Sheet

Share your Google Sheet with:

```
your-service-account@project.iam.gserviceaccount.com
```

Permission: **Editor**

---

## 📊 ETL Pipeline Details

### 1. Extract

* Reads CSV file
* Validates dataset size

```python
df = pd.read_csv("file.csv")
```

---

### 2. Transform

#### Key Transformations:

* Data type conversions
* Missing value handling
* Duplicate removal
* Outlier treatment (IQR)
* Feature engineering

#### Features Created:

| Feature         | Description         |
| --------------- | ------------------- |
| order_hour      | Extracted hour      |
| order_day       | Day name            |
| is_delayed      | Delivery > 40 min   |
| rating_category | High / Medium / Low |
| cancel_flag     | Binary cancellation |
| at_risk         | Combined risk       |
| risk_score      | Weighted score      |

---

### 3. Load

* Connect to Google Sheets
* Clear existing data
* Upload transformed dataset

---
# 🚀 Airflow ETL Pipeline with PostgreSQL & Google Sheets (Complete Guide)

---

## 🔐 Google Credentials Setup (CRITICAL SECTION)

This is the **most failure-prone part** of your entire pipeline. If this is wrong, your DAG will fail regardless of correct code.

---

## 🧭 Step-by-Step: Create `credentials.json`

### 1. Open Google Cloud Console

Go to:

```text
https://console.cloud.google.com/
```

---

### 2. Create New Project

* Click **Select Project → New Project**
* Name: `airflow-etl-project`
* Click **Create**

---

### 3. Enable Required APIs

Go to:

```text
APIs & Services → Library
```

Enable BOTH:

* Google Sheets API
* Google Drive API

👉 Click each → **Enable**

---

### 4. Create Service Account

Go to:

```text
APIs & Services → Credentials → Create Credentials → Service Account
```

Fill:

| Field | Value           |
| ----- | --------------- |
| Name  | airflow-service |
| ID    | auto            |
| Role  | Editor          |

Click **Done**

---

### 5. Generate JSON Key

* Click your service account
* Go to **Keys tab**
* Click **Add Key → Create New Key → JSON**

⬇️ File downloads automatically → **THIS is your `credentials.json`**

---

## 📂 Where to Place `credentials.json`

Put it here:

```bash
dags/credentials.json
```

Inside container path:

```python
"/opt/airflow/dags/credentials.json"
```

---

## 🔗 VERY IMPORTANT: Share Google Sheet

Copy service account email from JSON:

```json
"client_email": "airflow-service@project-id.iam.gserviceaccount.com"
```

Now:

1. Open your Google Sheet
2. Click **Share**
3. Paste email
4. Give **Editor access**

👉 If you skip this → **PERMISSION ERROR**

---

## 🧪 Test Connection (Inside DAG)

Add this in `load()`:

```python
print("Testing Google Sheets connection...")

spreadsheet = client.open_by_key("YOUR_SHEET_ID")
print(f"Connected to: {spreadsheet.title}")
```

---

## 🚨 WHERE TO SEE ERRORS (VERY IMPORTANT)

You should NEVER debug blindly. Always inspect logs.

---

## 1️⃣ Airflow UI Logs (PRIMARY SOURCE)

Go to:

```text
http://localhost:8080
```

Steps:

1. Click your DAG → `restaurant_etl_pipeline`
2. Click task (e.g., `load`)
3. Click **Logs**

---

### Example Errors

#### ❌ Credentials Not Found

```text
FileNotFoundError: credentials.json not found
```

👉 Fix:

* Wrong path
* File not inside container

---

#### ❌ Permission Error

```text
gspread.exceptions.APIError: 403 Permission denied
```

👉 Fix:

* Sheet not shared with service account

---

#### ❌ Invalid Spreadsheet ID

```text
gspread.exceptions.SpreadsheetNotFound
```

👉 Fix:

* Wrong `SPREADSHEET_ID`

---

---

## 2️⃣ Docker Logs

Get container ID:

```powershell
docker ps
```

Then:

```powershell
docker logs <airflow_container_id>
```

---

## 3️⃣ Inside Container Debugging

Enter container:

```powershell
docker exec -it <container_id> bash
```

Check files:

```bash
ls /opt/airflow/dags
```

Check credentials:

```bash
cat /opt/airflow/dags/credentials.json
```

---

## 🧠 Root Cause Mapping (Think Like Engineer)

| Error                 | Root Cause       | Fix                        |
| --------------------- | ---------------- | -------------------------- |
| 403 error             | Sheet not shared | Share with service account |
| File not found        | Wrong path       | Move JSON into dags        |
| Auth failed           | Invalid JSON     | Re-download key            |
| Spreadsheet not found | Wrong ID         | Copy correct ID            |

---

## ⚡ Pro-Level Debug Strategy

Instead of guessing:

1. Run DAG
2. Open logs
3. Identify **exact failure line**
4. Map error → root cause

👉 This is how real data engineers debug pipelines

---

## 🔥 Hard Truth

Most beginners fail not because of:

* Code ❌
* Logic ❌

But because of:

* Credentials setup
* Permissions
* Wrong paths

---

## 🧨 Upgrade Thinking

Don’t treat credentials as a file.

Treat them as:

```text
Access Control Layer
```

---

## ✔️ Final Checklist

Before running DAG:

* [ ] APIs enabled
* [ ] credentials.json downloaded
* [ ] File placed in `/dags`
* [ ] Sheet shared with service account
* [ ] Correct spreadsheet ID used
* [ ] Logs accessible

---

## 🚀 Outcome

If everything is correct:

```text
Airflow → Auth → Connect → Push → Success ✅
```

---



## 🔁 DAG Workflow

```python
t1 >> t2 >> t3
```

| Task      | Description     |
| --------- | --------------- |
| extract   | Load raw data   |
| transform | Clean & process |
| load      | Push to Sheets  |

---

## 🧪 Validation & Debugging

### Check Logs

```bash
docker logs <container_id>
```

---

### Enter Container

```bash
docker exec -it <container_id> bash
```

---

### Verify Files

```bash
ls /opt/airflow/dags
```

---

## 🚨 Common Errors & Fixes

### ❌ Google Sheets Not Updating

**Cause:**

* Sheet not shared

**Fix:**

* Share with service account

---

### ❌ Credentials Not Found

**Cause:**

* Wrong file path

**Fix:**

```
/opt/airflow/dags/credentials.json
```

---

### ❌ DAG Not Visible

**Cause:**

* File not in `dags/`

---

### ❌ Module Not Found

**Fix:**

```bash
pip install pandas gspread oauth2client
```

---

## ⚡ Performance Optimization

* Chunk processing (low memory usage)
* Vectorized operations
* IQR-based outlier handling
* Efficient CSV reading

---

## 🔥 Advanced Enhancements

### 1. PostgreSQL Data Storage

Instead of CSV:

```
Transform → PostgreSQL → Analytics
```

---

### 2. Real-Time Dashboard

Use:

* Google Looker Studio
* Power BI
* Tableau

---

### 3. Incremental Loading

Load only new data:

```
WHERE date > last_run
```

---

### 4. Scheduling

Current:

```
@daily
```

Options:

```
@hourly
@weekly
cron expressions
```

---

## 📈 Use Cases

* Food delivery analytics
* Logistics optimization
* Customer satisfaction analysis
* Risk prediction

---

## 🧨 Limitations

* Google Sheets is not scalable for large datasets
* CSV dependency
* No real-time ingestion

---

## 🚀 Future Roadmap

* Replace CSV with database ingestion
* Add Kafka for streaming
* Deploy on cloud (AWS/GCP)
* Add ML model for prediction

---

## 🧾 Key Learnings

* Airflow orchestration
* ETL pipeline design
* Data transformation techniques
* API integration
* Containerized workflows

---

## 🏁 Conclusion

This project demonstrates how to design a **modular, scalable ETL pipeline** using modern data engineering tools. It bridges the gap between raw data and actionable insights through structured processing and automation.

---

## 💡 Final Thought

This is not just a project.

It is a **foundation for building data platforms**.

Move from:

```
Scripts → Pipelines → Platforms
```

---

## 📎 Author

Developed as a practical implementation of **data engineering concepts using Airflow**.

---

