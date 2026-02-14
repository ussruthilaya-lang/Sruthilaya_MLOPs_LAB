# Airflow MLOps Lab - Customer Segmentation Pipeline

## 📋 Project Overview

This project demonstrates a complete **Machine Learning Operations (MLOps)** pipeline using **Apache Airflow** to orchestrate a customer segmentation workflow. The pipeline loads credit card customer data, preprocesses it, builds a K-Means clustering model, and evaluates the optimal number of clusters using the elbow method.

## 🎯 Learning Objectives

Through this project, you will learn:

1. **Workflow Orchestration**: How to use Apache Airflow to orchestrate multi-step ML pipelines
2. **Task Dependencies**: Managing dependencies between data loading, preprocessing, model training, and evaluation
3. **Docker Containerization**: Running Airflow in a containerized environment with Docker Compose
4. **XCom Communication**: Passing data between Airflow tasks using XCom (cross-communication)
5. **ML Pipeline Design**: Structuring a complete machine learning workflow from data to model evaluation
6. **K-Means Clustering**: Implementing unsupervised learning for customer segmentation
7. **Elbow Method**: Determining optimal cluster count using the elbow method

## 📁 Project Structure

```
Airflow_Lab/
├── dags/
│   └── airflow.py              # Main DAG definition with task orchestration
├── src/
│   ├── __init__.py             # Package initialization
│   └── lay.py                  # Core ML functions (load, preprocess, train, evaluate)
├── data/
│   ├── file.csv                # Training dataset (8952 customer records)
│   └── test.csv                # Test dataset for predictions
├── logs/                       # Airflow execution logs
├── plugins/                    # Custom Airflow plugins (if any)
├── working_data/               # Temporary working directory
├── docker-compose.yaml         # Docker Compose configuration for Airflow
├── setup.sh                    # Setup script for initializing Airflow
└── .env                        # Environment variables (AIRFLOW_UID)
```

## 🔧 File Descriptions

### 1. `dags/airflow.py` - DAG Definition
**Purpose**: Defines the Airflow DAG (Directed Acyclic Graph) that orchestrates the ML pipeline.

**Key Components**:
- **DAG Configuration**: Named `Airflow_Lab1`, runs with CeleryExecutor
- **4 Sequential Tasks**:
  1. `load_data_task` - Loads customer data from CSV
  2. `data_preprocessing_task` - Cleans and normalizes data
  3. `build_save_model_task` - Trains K-Means models and saves the best one
  4. `load_model_task` - Loads model and evaluates using elbow method

**Task Dependencies**: `load_data → preprocess → train → evaluate`

### 2. `src/lay.py` - ML Pipeline Functions
**Purpose**: Contains the core machine learning logic called by Airflow tasks.

**Functions**:

#### `load_data()`
- Reads `data/file.csv` containing 8,952 customer records
- Serializes DataFrame using pickle and base64 encoding for XCom compatibility
- Returns JSON-safe encoded data

#### `data_preprocessing(data_b64)`
- Deserializes input data from XCom
- Removes null values
- Selects features: `BALANCE`, `PURCHASES`, `CREDIT_LIMIT`
- Applies MinMax scaling (0-1 normalization)
- Returns serialized preprocessed data

#### `build_save_model(data_b64, filename)`
- Trains K-Means models for k=1 to k=49 clusters
- Calculates SSE (Sum of Squared Errors) for each k
- Saves the final model (k=49) to `model/model.sav`
- Returns SSE list for elbow analysis

#### `load_model_elbow(filename, sse)`
- Loads saved model from `model/model.sav`
- Uses KneeLocator to find optimal k using elbow method
- Makes predictions on `data/test.csv`
- Returns first prediction as integer

### 3. `data/file.csv` - Training Dataset
**Purpose**: Main training dataset for customer segmentation.

**Features** (18 columns):
- `CUST_ID`: Customer ID
- `BALANCE`: Account balance
- `PURCHASES`: Total purchase amount
- `CREDIT_LIMIT`: Credit card limit
- `CASH_ADVANCE`: Cash advance amount
- And 13 other behavioral features

**Size**: 8,952 customer records

### 4. `data/test.csv` - Test Dataset
**Purpose**: Single test record for model prediction validation.

**Content**: One customer with features: `BALANCE=3202.47`, `PURCHASES=124.5`, `CREDIT_LIMIT=241514`

### 5. `docker-compose.yaml` - Airflow Infrastructure
**Purpose**: Defines the complete Airflow stack using Docker containers.

**Services**:
- **postgres**: PostgreSQL database for Airflow metadata
- **redis**: Message broker for Celery executor
- **airflow-webserver**: Web UI (port 8080)
- **airflow-scheduler**: Task scheduler
- **airflow-worker**: Celery worker for task execution
- **airflow-triggerer**: Handles deferred tasks
- **airflow-init**: Initialization service

**Configuration**:
- Executor: CeleryExecutor (distributed task execution)
- Image: `apache/airflow:2.5.1`
- Volumes: Maps `dags/`, `logs/`, `plugins/` directories

### 6. `setup.sh` - Initialization Script
**Purpose**: Prepares the environment for Airflow.

**Actions**:
1. Removes old `.env` file and directories
2. Stops and removes existing Docker containers
3. Creates fresh `logs/`, `plugins/`, `config/` directories
4. Sets `AIRFLOW_UID=50000` in `.env`
5. Runs Airflow config list command

## 🚀 How to Run the Project

### Prerequisites
- Docker Desktop installed and running
- At least 4GB RAM available
- At least 10GB disk space

### Step 1: Initial Setup
```bash
# Navigate to project directory
cd d:\Sruthi\NEU\MLOPs\Sruthi_Labs\Sruthilaya_MLOPs_LAB\Labs\Airflow_Lab

# Run setup script (Linux/Mac)
bash setup.sh

# Or manually on Windows PowerShell:
Remove-Item .env -ErrorAction SilentlyContinue
docker compose down -v
New-Item -ItemType Directory -Force -Path logs, plugins, config
"AIRFLOW_UID=50000" | Out-File -FilePath .env -Encoding ASCII
```

### Step 2: Start Airflow
```bash
# Start all Airflow services
docker compose up

# Or run in detached mode (background)
docker compose up -d
```

**Wait for initialization** (2-3 minutes). You'll see logs indicating services are healthy.

### Step 3: Access Airflow Web UI
1. Open browser: `http://localhost:8080`
2. Login credentials:
   - Username: `airflow`
   - Password: `airflow`

### Step 4: Run the DAG
1. In the Airflow UI, find the DAG named `Airflow_Lab1`
2. Toggle the DAG to "ON" (unpause it)
3. Click the "Play" button to trigger a manual run
4. Monitor task execution in the Graph or Grid view

### Step 5: View Results
- **Logs**: Click on individual tasks to view execution logs
- **Model**: Check `model/model.sav` for the saved K-Means model
- **Optimal Clusters**: View scheduler logs for the elbow method output

### Step 6: Stop Airflow
```bash
# Stop all services
docker compose down

# Stop and remove volumes (clean slate)
docker compose down -v
```

## 📊 What the Pipeline Does

### Data Flow

```
1. Load Data (file.csv)
   ↓ [8,952 customer records]
   
2. Preprocess
   ↓ [Select BALANCE, PURCHASES, CREDIT_LIMIT]
   ↓ [Remove nulls, MinMax scale to 0-1]
   
3. Train K-Means Models
   ↓ [Train k=1 to k=49]
   ↓ [Calculate SSE for each k]
   ↓ [Save model (k=49)]
   
4. Evaluate & Predict
   ↓ [Find optimal k using elbow method]
   ↓ [Predict cluster for test.csv]
   ↓ [Output: cluster assignment]
```

### Expected Output

When the pipeline completes successfully:

1. **Model File**: `model/model.sav` - Pickled K-Means model
2. **Optimal Clusters**: Printed in logs (typically 6-8 clusters for this dataset)
3. **Test Prediction**: Cluster assignment for the test customer
4. **Task Status**: All 4 tasks show "success" in Airflow UI

## 🧠 Key Concepts Learned

### 1. Airflow DAG Structure
- **DAG**: Directed Acyclic Graph defining workflow
- **Tasks**: Individual units of work (PythonOperator)
- **Dependencies**: `>>` operator chains tasks sequentially
- **XCom**: Cross-task communication for passing data

### 2. ML Pipeline Components
- **Data Loading**: Reading CSV files into pandas DataFrames
- **Preprocessing**: Feature selection and normalization
- **Model Training**: Iterative K-Means clustering
- **Model Evaluation**: Elbow method for optimal k selection

### 3. Docker Orchestration
- **Multi-container setup**: Postgres, Redis, Airflow components
- **Service dependencies**: Ensures proper startup order
- **Volume mounting**: Shares code and data with containers
- **Health checks**: Monitors service availability

### 4. Data Serialization
- **Pickle**: Python object serialization
- **Base64**: Binary-to-text encoding for JSON compatibility
- **XCom limitations**: Requires JSON-safe data types

## 🔍 Troubleshooting

### Issue: Containers won't start
**Solution**: Ensure Docker Desktop is running and has sufficient resources (4GB+ RAM)

### Issue: Port 8080 already in use
**Solution**: Stop other services using port 8080 or modify `docker-compose.yaml`

### Issue: Tasks fail with import errors
**Solution**: Ensure `src/` directory is properly mounted and `__init__.py` exists

### Issue: XCom serialization errors
**Solution**: Verify data is being encoded/decoded correctly with base64 and pickle

### Issue: Model file not found
**Solution**: Check that `model/` directory exists and has write permissions

## 📚 Additional Resources

- [Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [K-Means Clustering](https://scikit-learn.org/stable/modules/clustering.html#k-means)
- [Elbow Method](https://en.wikipedia.org/wiki/Elbow_method_(clustering))
- [Docker Compose](https://docs.docker.com/compose/)

## 👥 Credits

- **Owner**: Ramin Mohammadi
- **Course**: MLOps Lab Series
- **Institution**: Northeastern University

## 📝 License

This project is for educational purposes as part of the MLOps course curriculum.
