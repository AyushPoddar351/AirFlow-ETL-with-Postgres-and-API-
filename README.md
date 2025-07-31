# Apache Airflow ETL Pipeline - NASA APOD Data Engineering

A production-ready ETL (Extract, Transform, Load) pipeline built with Apache Airflow that automatically fetches NASA's Astronomy Picture of the Day (APOD) data, processes it, and stores it in PostgreSQL database with enterprise-grade orchestration and monitoring.

## 🎯 Project Overview

This project demonstrates advanced data engineering capabilities using Apache Airflow for workflow orchestration. The pipeline extracts daily astronomical data from NASA's public API, transforms it for analytics, and loads it into a PostgreSQL database, showcasing modern data pipeline architecture and best practices.

## 🚀 Key Features

### Data Engineering Architecture
- **Apache Airflow**: Enterprise workflow orchestration platform
- **ETL Pipeline**: Extract-Transform-Load data processing
- **Task Dependencies**: Automated workflow management
- **Error Handling**: Robust retry mechanisms and failure recovery
- **Scheduling**: Daily automated data ingestion

### Data Sources & Storage
- **NASA APOD API**: Astronomy Picture of the Day data source
- **PostgreSQL Database**: Structured data storage
- **RESTful Integration**: HTTP-based API consumption
- **Data Validation**: Comprehensive data quality checks

### DevOps & Infrastructure
- **Docker Containerization**: Portable deployment environment
- **Astronomer Runtime**: Cloud-native Airflow distribution
- **Database Connection Management**: Secure credential handling
- **Environment Configuration**: Scalable infrastructure setup

## 📊 Pipeline Architecture

### ETL Workflow
```
NASA API → Extract → Transform → PostgreSQL
    ↓         ↓         ↓          ↓
  APOD Data  JSON     Structured  Database
             Response Processing  Storage
```

### Task Flow
1. **Create Table**: Initialize PostgreSQL schema
2. **Extract APOD**: Fetch data from NASA API
3. **Transform APOD**: Process and validate JSON response
4. **Load APOD**: Insert transformed data into database

## 🏗️ Project Structure

```
etl-airflow-project/
├── .astro/                        # Astronomer CLI configuration
│   ├── config.yaml               # Project settings
│   └── test_dag_integrity_default.py  # DAG validation tests
├── dags/
│   ├── etl.py                    # Main NASA APOD ETL pipeline
│   └── exampledag.py             # Astronaut data example DAG
├── tests/
│   └── dags/
│       └── test_dag_example.py   # Comprehensive DAG testing
├── docker-compose.yml            # PostgreSQL service configuration
├── Dockerfile                    # Airflow runtime container
├── requirements.txt              # Python dependencies
└── README.md                     # Project documentation
```

## 🛠️ Installation & Setup

### Prerequisites
- Docker & Docker Compose
- Astronomer CLI (astro)
- Python 3.8+
- PostgreSQL client (optional)

### 1. Clone Repository
```bash
git clone <repository-url>
cd etl-airflow-project
```

### 2. Install Astronomer CLI
```bash
# macOS
brew install astro

# Linux/Windows
curl -sSL install.astronomer.io | sudo bash
```

### 3. Start Services
```bash
# Start PostgreSQL database
docker-compose up -d postgres

# Initialize Airflow project
astro dev init

# Start Airflow development environment
astro dev start
```

### 4. Access Airflow UI
- **URL**: http://localhost:8080
- **Username**: admin
- **Password**: admin

## ⚙️ Configuration

### Database Connection (my_postgres_connection)
```
Connection Type: Postgres
Host: postgres_db
Schema: postgres
Login: postgres
Password: postgres
Port: 5432
```

### NASA API Connection (nasa_api)
```
Connection Type: HTTP
Host: https://api.nasa.gov
Extra: {"api_key": "YOUR_NASA_API_KEY"}
```

### Environment Variables
```bash
# NASA API Key (get from https://api.nasa.gov/)
NASA_API_KEY=your_api_key_here

# Database credentials
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres
```

## 📈 ETL Pipeline Details

### Main DAG: `nasa_apod_postgres`

#### Task 1: Create Table
```python
@task
def create_table():
    postgres_hook = PostgresHook(postgres_conn_id="my_postgres_connection")
    create_table_query = """
        CREATE TABLE IF NOT EXISTS apod_data (
            id SERIAL PRIMARY KEY,
            title VARCHAR(255),
            explanation TEXT,
            url TEXT,
            date DATE,
            media_type VARCHAR(50)
        );
    """
    postgres_hook.run(create_table_query)
```

#### Task 2: Extract APOD Data
```python
extract_apod = HttpOperator(
    task_id="extract_apod",
    http_conn_id="nasa_api",
    endpoint="/planetary/apod",
    method="GET",
    data={"api_key": "{{ conn.nasa_api.extra_dejson.api_key }}"},
    response_filter=lambda response: response.json()
)
```

#### Task 3: Transform Data
```python
@task
def transform_apod(response):
    apod_data = {
        'title': response.get('title', ''),
        'explanation': response.get('explanation', ''),
        'url': response.get('url', ''),
        'date': response.get('date', ''),
        'media_type': response.get('media_type', '')
    }
    return apod_data
```

#### Task 4: Load to Database
```python
@task
def load_apod(apod_data):
    postgres_hook = PostgresHook(postgres_conn_id="my_postgres_connection")
    insert_query = """
        INSERT INTO apod_data (title, explanation, url, date, media_type)
        VALUES (%s, %s, %s, %s, %s);
    """
    postgres_hook.run(insert_query, parameters=(
        apod_data['title'], 
        apod_data['explanation'], 
        apod_data['url'], 
        apod_data['date'], 
        apod_data['media_type']
    ))
```

## 🔧 Technical Stack

### Workflow Orchestration
- **Apache Airflow 2.8+**: Modern workflow management platform
- **Astronomer Runtime**: Enterprise Airflow distribution
- **TaskFlow API**: Pythonic task definition and dependency management
- **XCom**: Inter-task communication and data passing

### Data Processing
- **HTTP Operator**: REST API integration
- **PostgreSQL Hook**: Database connectivity and operations
- **Python Tasks**: Custom transformation logic
- **JSON Processing**: Structured data handling

### Infrastructure
- **Docker**: Containerized application deployment
- **PostgreSQL 13**: Relational database management
- **Docker Compose**: Multi-service orchestration
- **Astronomer CLI**: Development and deployment tooling

## 📊 Data Schema

### APOD Data Table Structure
```sql
CREATE TABLE apod_data (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),           -- Daily image/video title
    explanation TEXT,             -- Detailed astronomical description  
    url TEXT,                     -- Media file URL
    date DATE,                    -- Publication date
    media_type VARCHAR(50)        -- 'image' or 'video'
);
```

### Sample Data
```json
{
    "title": "The Horsehead Nebula",
    "explanation": "The Horsehead Nebula is one of the most...",
    "url": "https://apod.nasa.gov/apod/image/2024/horsehead.jpg",
    "date": "2024-01-15",
    "media_type": "image"
}
```

## 🔄 Pipeline Execution

### Scheduling
- **Frequency**: Daily execution (`@daily`)
- **Start Date**: Current date minus 1 day
- **Catchup**: Disabled for real-time processing
- **Timezone**: UTC by default

### Task Dependencies
```python
create_table() >> extract_apod 
api_response = extract_apod.output
transformed_data = transform_apod(api_response)
load_apod(transformed_data)
```

### Execution Flow
1. **Table Creation** → **API Extraction** (sequential)
2. **Data Transformation** ← **API Response** (data-driven)
3. **Database Loading** ← **Transformed Data** (data-driven)

## 🧪 Testing & Quality Assurance

### DAG Integrity Tests
```python
# Automated tests for:
- Import error detection
- Task dependency validation
- Connection configuration
- Data type validation
```

### Test Execution
```bash
# Run DAG integrity tests
astro dev pytest tests/

# Validate DAG parsing
astro dev parse

# Test specific DAG
airflow dags test nasa_apod_postgres 2024-01-15
```

### Quality Checks
- **Retry Logic**: 3 automatic retries per task
- **Error Handling**: Comprehensive exception management
- **Data Validation**: Schema compliance verification
- **Connection Testing**: Database and API connectivity

## 📈 Monitoring & Observability

### Airflow UI Features
- **DAG Graph View**: Visual pipeline representation
- **Task Instance Logs**: Detailed execution tracking
- **Connection Management**: Secure credential storage
- **Scheduler Monitoring**: Pipeline health dashboards

### Performance Metrics
- **Task Duration**: Execution time tracking
- **Success Rate**: Pipeline reliability metrics
- **Data Volume**: Daily ingestion statistics
- **Error Patterns**: Failure analysis and trends

## 🚀 Deployment Options

### Local Development
```bash
astro dev start    # Start local Airflow
astro dev stop     # Stop local environment
astro dev restart  # Restart with new changes
```

### Production Deployment
```bash
# Deploy to Astronomer Cloud
astro deploy

# Deploy to custom Kubernetes cluster
astro deploy --deployment-id <deployment-id>
```

### Docker Production
```bash
# Build production image
docker build -t nasa-etl-pipeline .

# Run with docker-compose
docker-compose up -d
```

## 🔧 Advanced Configuration

### Custom Connections
```python
# Programmatic connection creation
from airflow.models import Connection

nasa_conn = Connection(
    conn_id='nasa_api',
    conn_type='HTTP',
    host='https://api.nasa.gov',
    extra='{"api_key": "your_key_here"}'
)
```

### Dynamic Task Generation
```python
# Example: Process multiple NASA APIs
@task
def process_multiple_endpoints():
    endpoints = ['/planetary/apod', '/mars-photos/api/v1/rovers']
    return [process_endpoint(ep) for ep in endpoints]
```

### Error Handling & Alerting
```python
# Custom failure callbacks
def task_failure_alert(context):
    # Send Slack/email notifications
    pass

default_args = {
    'owner': 'data-engineering',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'on_failure_callback': task_failure_alert
}
```

## 📊 Example DAG: Astronaut Tracker

### Dynamic Task Mapping
```python
@task
def get_astronauts() -> list[dict]:
    """Fetch current astronauts in space"""
    r = requests.get("http://api.open-notify.org/astros.json")
    return r.json()["people"]

# Dynamic task creation based on API response
print_astronaut_craft.partial(greeting="Hello! :)").expand(
    person_in_space=get_astronauts()
)
```

## 🔄 Future Enhancements

### Pipeline Improvements
- **Data Lake Integration**: S3/MinIO storage layer
- **Stream Processing**: Real-time data ingestion
- **Data Quality**: Great Expectations integration
- **Lineage Tracking**: Apache Atlas metadata management

### Advanced Features
- **Machine Learning**: Automated image classification
- **API Rate Limiting**: Intelligent request throttling
- **Multi-source Integration**: Additional NASA APIs
- **Data Cataloging**: Searchable metadata repository

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/enhancement`)
3. Implement changes with tests
4. Run DAG integrity tests (`astro dev pytest`)
5. Commit changes (`git commit -m 'Add enhancement'`)
6. Push to branch (`git push origin feature/enhancement`)
7. Create Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👤 Author

**Ayush Poddar**
- Email: ayushpoddar351@gmail.com
- LinkedIn: [Your LinkedIn Profile]
- GitHub: [Your GitHub Profile]

## 🙏 Acknowledgments

- **NASA Open Data**: Public API access and astronomical content
- **Apache Airflow Community**: Workflow orchestration framework
- **Astronomer**: Cloud-native Airflow platform and tooling
- **PostgreSQL Team**: Robust database management system
- **Docker Community**: Containerization technology

## 📚 Key Learning Outcomes

This project demonstrates:
- **Enterprise Data Engineering**: Production-ready ETL pipelines
- **Workflow Orchestration**: Apache Airflow mastery
- **API Integration**: RESTful service consumption
- **Database Operations**: PostgreSQL data management
- **DevOps Practices**: Containerization and CI/CD
- **Data Quality**: Testing and monitoring implementation
- **Cloud Technologies**: Modern data stack integration

---

*This project showcases modern data engineering practices using Apache Airflow for automated data pipeline orchestration, combining NASA's astronomical data with enterprise-grade workflow management for scalable data processing solutions.*
