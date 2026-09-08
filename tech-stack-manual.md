# Tech Stack Manual

## 1. Run Questions

What config files exist in the s?


### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
|.env|root|POSTGRES_DB|Database name|Tells system which Postgre Database to look at  |
|.env |root |POSTGRES_USER |Username|Tells system which username to use for login |
|.env |root |POSTGRES_PASSWORD|Password|Tells system which password to use for login |
|.env |root |DATA_SOURCE_NAME|Full name for Data Source|Pieces together strings from other configs for the full link to database||
|Makefile|root|setup|sets up the system for a new user|Runs bash script to install everything needed to run the system|
|Makefile |root |up |Starts the system |Runs docker commands to get the system running |
|Makefile |root |up-api |Starts the API |Runs docker commands to only start the API |
|docker-compose.yml|root|database|configuration for database|services, settings, and rules for the database|
|docker-compose.yml|root|api|configuration for api|services, settings, and rules for the api|
|docker-compose.yml|root|client|configuration for client|services, settings, and rules for the client|


### 1b. How to Start It
 make up - starts the system

 make up - api - starts the api

 make up - client-api starts api and client 


### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
|database |"5433:5432" | postgresql://learnops:learnops123@database:5432/learningplatform?sslmode=disable |
|api | "8000:8000"| localhost:8000 |
|client | "3000:3000"| localhost:3000 |

### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| api | database | API needs the data from the db to perform its functions |
| client | api | Prometheus scrapes metrics from api service|
| prometheus  (client) | api | Prometheus scrapes metrics from api service|
| grafana  (client) | prometheus | creates visualizations from prometheus data |
| postgres_exporter  (client) | database | exports postgre data |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| api |entrypoint.sh |/LearningPlatform/urls.py |
| client | Dockerfile | /src.index.js |

---

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| api | | |
| client | | |

---

## 3. System Overview
