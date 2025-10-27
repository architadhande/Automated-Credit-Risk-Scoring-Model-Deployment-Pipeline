# Automated Credit Risk Scoring & Model Deployment Pipeline

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104.1-009688.svg)](https://fastapi.tiangolo.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A production-ready end-to-end machine learning pipeline for predicting credit default probabilities with real-time scoring capabilities. Built with Python, integrated with GCP and Snowflake, orchestrated via Apache Airflow.

## Project Overview

This system predicts client credit default probabilities using demographic, transactional, and macroeconomic indicators. It features automated daily data ingestion, model retraining, and real-time API-based predictions.

**Key Achievements:**
- ✅ 45% improvement in scoring efficiency (real-time predictions < 100ms)
- ✅ AUC-ROC: 0.82+ | Precision: 0.78+ | Recall: 0.75+
- ✅ 99.9% API uptime with auto-scaling
- ✅ Daily automated data refresh with validation

## Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐      ┌──────────────┐
│   Source    │ ───▶ │  Data Lake   │ ───▶ │ Data Warehouse│───▶│ ML Pipeline  │
│  Systems    │      │  (GCS/BQ)    │      │  (Snowflake)  │    │ (Training)   │
└─────────────┘      └──────────────┘      └─────────────┘      └──────────────┘
                                                                         │
                                                                         ▼
┌─────────────┐      ┌──────────────┐      ┌─────────────┐      ┌──────────────┐
│   Client    │ ◀─── │  REST API    │ ◀─── │   Model     │ ◀─── │   MLflow     │
│ Application │      │  (FastAPI)   │      │  Registry   │      │  Tracking    │
└─────────────┘      └──────────────┘      └─────────────┘      └──────────────┘
```

## Quick Start

### Prerequisites

- Python 3.9+
- Docker & Docker Compose
- GCP Account with BigQuery & GCS enabled
- Snowflake Account
- Apache Airflow 2.7+

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/credit-risk-ml-pipeline.git
cd credit-risk-ml-pipeline

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment template
cp .env.example .env
# Edit .env with your credentials
```

### Configuration

1. **Set up GCP credentials:**
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```

2. **Configure Snowflake connection in `.env`:**
```env
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_USER=your_user
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_DATABASE=CREDIT_RISK_DB
SNOWFLAKE_SCHEMA=PROD
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
```

3. **Set up database schemas:**
```bash
python scripts/setup_database.py
```

### Running Locally

```bash
# Run tests
pytest tests/ -v

# Start the API server
uvicorn src.api.main:app --reload --host 0.0.0.0 --port 8000

# Access API documentation
# Open browser: http://localhost:8000/docs
```

### Using Docker

```bash
# Build and run with docker-compose
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

## Project Structure

```
credit-risk-ml-pipeline/
├── src/
│   ├── data_ingestion/          # ETL pipelines
│   ├── preprocessing/            # Data cleaning & validation
│   ├── feature_engineering/      # Feature creation
│   ├── models/                   # ML models (LR, XGBoost)
│   ├── api/                      # FastAPI application
│   └── utils/                    # Helper functions
├── airflow/
│   └── dags/                     # Airflow DAGs
├── notebooks/                    # Jupyter notebooks
├── tests/                        # Unit & integration tests
├── deployment/                   # Docker & K8s configs
├── config/                       # Configuration files
└── docs/                         # Documentation
```

## Pipeline Components

### 1. Data Ingestion (Daily 2 AM UTC)
- Pulls data from CRM, core banking systems, and external APIs
- Validates data quality and completeness
- Stores raw data in GCS and loads to Snowflake

### 2. Feature Engineering (Daily 4 AM UTC)
- **Demographic Features:** Age, income, employment status, education
- **Transactional Features:** Payment history, credit utilization, account age
- **Macroeconomic Features:** Interest rates, unemployment rate, GDP growth

### 3. Model Training (Weekly - Sunday 1 AM UTC)
- Trains Logistic Regression and XGBoost models
- Performs cross-validation and hyperparameter tuning
- Validates performance against holdout set
- Registers models in MLflow if performance improves

### 4. Model Serving (24/7)
- REST API for real-time predictions
- Batch prediction endpoint
- Model versioning and A/B testing support

## API Usage

### Single Prediction

```python
import requests

url = "http://localhost:8000/api/v1/predict"
payload = {
    "customer_id": "CUST_12345",
    "age": 35,
    "annual_income": 75000,
    "credit_utilization": 0.35,
    "payment_history_score": 720,
    "employment_years": 8,
    "num_credit_accounts": 5,
    "total_debt": 25000,
    "interest_rate": 0.065,
    "unemployment_rate": 0.04
}

response = requests.post(url, json=payload)
print(response.json())
# Output: {"default_probability": 0.15, "risk_category": "low", "model_version": "v2.3.1"}
```

### Batch Prediction

```python
url = "http://localhost:8000/api/v1/predict/batch"
payload = {
    "customers": [
        {"customer_id": "CUST_001", "age": 28, ...},
        {"customer_id": "CUST_002", "age": 45, ...}
    ]
}

response = requests.post(url, json=payload)
```

## Model Performance

### Current Production Model (XGBoost v2.3.1)

| Metric | Train | Validation | Test |
|--------|-------|------------|------|
| AUC-ROC | 0.847 | 0.823 | 0.819 |
| Precision | 0.812 | 0.783 | 0.778 |
| Recall | 0.791 | 0.756 | 0.752 |
| F1-Score | 0.801 | 0.769 | 0.765 |
| Accuracy | 0.876 | 0.864 | 0.861 |

### Feature Importance (Top 10)

1. Payment History Score (0.18)
2. Credit Utilization Ratio (0.15)
3. Total Debt to Income Ratio (0.12)
4. Number of Delinquencies (0.11)
5. Employment Years (0.09)
6. Annual Income (0.08)
7. Age (0.07)
8. Number of Credit Accounts (0.06)
9. Recent Credit Inquiries (0.05)
10. Interest Rate Environment (0.04)

## 🛠️ Technology Stack

### Data & Storage
- **Cloud Platform:** Google Cloud Platform (GCS, BigQuery)
- **Data Warehouse:** Snowflake
- **Object Storage:** Google Cloud Storage

### ML & Processing
- **ML Libraries:** scikit-learn, XGBoost, LightGBM
- **Data Processing:** Pandas, NumPy, Polars
- **Feature Store:** Feast (optional)
- **Experiment Tracking:** MLflow

### API & Deployment
- **API Framework:** FastAPI
- **Container:** Docker
- **Orchestration:** Kubernetes / Cloud Run
- **CI/CD:** GitHub Actions / Bitbucket Pipelines
- **Workflow Orchestration:** Apache Airflow

### Monitoring
- **Metrics:** Prometheus
- **Visualization:** Grafana
- **Logging:** Google Cloud Logging
- **Alerting:** PagerDuty / Slack

## Airflow DAGs

### 1. `daily_ingestion_dag`
Runs daily at 2 AM UTC
- Extract data from source systems
- Validate data quality
- Load to data warehouse

### 2. `feature_engineering_dag`
Runs daily at 4 AM UTC (after ingestion)
- Calculate demographic features
- Compute transactional metrics
- Fetch macroeconomic indicators
- Create feature table

### 3. `model_training_dag`
Runs weekly on Sunday at 1 AM UTC
- Load training data
- Train multiple model candidates
- Validate on holdout set
- Register best model in MLflow
- Deploy if performance improves

### 4. `model_monitoring_dag`
Runs daily at 6 AM UTC
- Monitor prediction distribution
- Check for data drift
- Calculate business metrics
- Send alerts if anomalies detected

## Deployment

### Deploy to GCP Cloud Run

```bash
# Build and push Docker image
docker build -t gcr.io/YOUR_PROJECT/credit-risk-api:latest -f deployment/docker/Dockerfile.api .
docker push gcr.io/YOUR_PROJECT/credit-risk-api:latest

# Deploy to Cloud Run
gcloud run deploy credit-risk-api \
  --image gcr.io/YOUR_PROJECT/credit-risk-api:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars "PROJECT_ID=YOUR_PROJECT"
```

### Deploy to Kubernetes

```bash
# Apply Kubernetes configurations
kubectl apply -f deployment/k8s/api-deployment.yaml
kubectl apply -f deployment/k8s/api-service.yaml
kubectl apply -f deployment/k8s/ingress.yaml

# Check deployment status
kubectl get pods -l app=credit-risk-api
kubectl get svc credit-risk-api
```

### Setup Airflow

```bash
# Initialize Airflow database
airflow db init

# Create admin user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com

# Configure connections
airflow connections add 'gcp_default' \
    --conn-type 'google_cloud_platform' \
    --conn-extra '{"extra__google_cloud_platform__project": "YOUR_PROJECT"}'

airflow connections add 'snowflake_default' \
    --conn-type 'snowflake' \
    --conn-host 'YOUR_ACCOUNT.snowflakecomputing.com' \
    --conn-login 'YOUR_USER' \
    --conn-password 'YOUR_PASSWORD' \
    --conn-schema 'PROD' \
    --conn-extra '{"database": "CREDIT_RISK_DB", "warehouse": "COMPUTE_WH"}'

# Start Airflow
airflow webserver --port 8080 &
airflow scheduler &
```

## Monitoring & Alerting

### Key Metrics to Monitor

1. **Data Quality Metrics**
   - Row counts and null percentages
   - Feature distribution shifts
   - Schema validation failures

2. **Model Performance Metrics**
   - Daily prediction distribution
   - AUC-ROC on recent data
   - Precision/Recall trends

3. **API Metrics**
   - Request latency (p50, p95, p99)
   - Throughput (requests/second)
   - Error rates by endpoint

4. **Business Metrics**
   - Approval rates by risk category
   - Default rates over time
   - Portfolio risk distribution

### Setting Up Alerts

Configure alerts in `config/alerts.yaml`:

```yaml
alerts:
  - name: high_api_latency
    metric: api_latency_p95
    threshold: 200  # ms
    duration: 5m
    severity: warning
    
  - name: model_performance_degradation
    metric: auc_roc
    threshold: 0.75
    comparison: less_than
    duration: 1d
    severity: critical
    
  - name: data_drift_detected
    metric: feature_drift_score
    threshold: 0.15
    duration: 1h
    severity: warning
```

## Testing

```bash
# Run all tests
pytest tests/ -v

# Run specific test categories
pytest tests/test_models.py -v
pytest tests/test_api.py -v

# Run with coverage
pytest --cov=src --cov-report=html

# Integration tests
pytest tests/integration/ -v
```

## Documentation

- [Architecture Overview](docs/architecture.md)
- [API Documentation](docs/api_documentation.md)
- [Deployment Guide](docs/deployment_guide.md)
- [Model Development Guide](docs/model_development.md)
- [Troubleshooting](docs/troubleshooting.md)

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


## 🗺️ Roadmap

- [ ] Add SHAP explanations for model predictions
- [ ] Implement automated A/B testing framework
- [ ] Add support for real-time feature computation
- [ ] Integrate with fraud detection system
- [ ] Build customer-facing dashboard
- [ ] Add support for alternative data sources
