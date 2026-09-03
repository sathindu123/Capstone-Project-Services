# 🧠 Backend Services

Core Business Logic Super-Repository for the Capstone Microservices Architecture — Student, Program, and Enrollment Management.

## 📋 Student & Submission Details

| Field | Details |
|---|---|
| **Student Name** | Sathindu Sathsara Kumara |
| **Student Number** | 241711053 |
| **GCP Project ID** | sathindu-gcp-lab |
| **Submission Type** | Alternative Option (Capstone Project) |

## 📖 Project Description

This repository houses the core business-logic microservices of the Capstone platform. Structured as a super-repository, it contains three independently deployable Git submodules that manage the fundamental academic domain entities: students, programs, and enrollments.

Each service registers itself with the Eureka service-registry (from the Backend Microservices Platform repo), fetches its configuration dynamically from the `config-server`, and runs on a randomly assigned port (`server.port=0`) to support horizontal scaling within a GCP Compute Engine Managed Instance Group (MIG). Process lifecycle on each VM instance is handled by PM2.

## 🧩 Included Submodules

| # | Submodule | Port | Database | Responsibility |
|---|---|---|---|---|
| 1️⃣ | `student-service` | Random (0) | PostgreSQL | Manages student records. Integrates with Google Cloud Storage (GCS) to dynamically upload and retrieve student profile pictures. |
| 2️⃣ | `program-service` | Random (0) | PostgreSQL | Manages academic program definitions and metadata. |
| 3️⃣ | `enrollment-service`| Random (0) | MongoDB | Manages student-to-program enrollment records using a non-relational data model. |

## 🏛️ High-Level Architecture

```text
                          ┌───────────────────────────────┐
                          │        api-gateway            │
                          │ (from Microservices Platform) │
                          └────────────────┬──────────────┘
                                           │  lb://
             ┌─────────────────────────────┼─────────────────────────────┐
             ▼                             ▼                             ▼
 ┌────────────────────────┐  ┌────────────────────────┐  ┌────────────────────────┐
 │   student-service      │  │   program-service      │  │   enrollment-service   │
 │   (Random Port)        │  │   (Random Port)        │  │   (Random Port)        │
 │                        │  │                        │  │                        │
 │ ┌────────────────────┐ │  │ ┌────────────────────┐ │  │ ┌────────────────────┐ │
 │ │ PostgreSQL         │ │  │ │ PostgreSQL         │ │  │ │ MongoDB            │ │
 │ └────────────────────┘ │  │ └────────────────────┘ │  │ └────────────────────┘ │
 │ ┌────────────────────┐ │  └────────────────────────┘  └────────────────────────┘
 │ │ Google Cloud Storage││
 │ │ (Profile Pictures)  ││
 │ └────────────────────┘ │
 └───────────┬────────────┘
             │
             ▼
 ┌────────────────────────┐
 │ Eureka Service Registry│
 │ + Config Server        │
 └────────────────────────┘
```

## 📁 Repository Structure

```text
backend-services/
├── student-service/         # Git Submodule — Random Port, PostgreSQL + GCS
├── program-service/         # Git Submodule — Random Port, PostgreSQL
├── enrollment-service/      # Git Submodule — Random Port, MongoDB
├── .gitmodules              # Submodule mapping definitions
└── README.md
```

## 🛠️ Technology Stack

| Category | Technology |
|---|---|
| **Language** | Java 17 |
| **Framework** | Spring Boot 3.3.2 |
| **Microservices Framework**| Spring Cloud 2023.0.3 |
| **Relational Persistence** | Spring Data JPA (PostgreSQL) |
| **Non-Relational Persistence**| Spring Data MongoDB |
| **Cloud Storage** | GCP Cloud Storage Spring Boot Starter |
| **Build Tool** | Maven |
| **Process Manager** | PM2 |
| **Cloud Provider** | Google Cloud Platform (GCP) |
| **Compute** | Compute Engine — Managed Instance Groups (MIGs) |
| **Object Storage** | Google Cloud Storage (GCS Bucket) |
| **Version Control** | Git (Submodule-based Super-Repo) |

## 🚀 Local Setup / Getting Started

### ✅ Prerequisites
* Java 17 (JDK)
* Maven 3.9+
* Git
* PostgreSQL instance (local or Cloud SQL)
* MongoDB instance (local or Atlas / Cloud-hosted)
* A GCP Service Account key with Storage Object Admin permissions (for `student-service`)
* The `config-server` and `service-registry` from the *Backend Microservices Platform* repo running locally.

### 1️⃣ Clone the Super-Repository with Submodules
```bash
git clone --recurse-submodules https://github.com/<your-username>/backend-services.git
cd backend-services
```
Already cloned without submodules? Run:
```bash
git submodule update --init --recursive
```

### 2️⃣ Configure Local Environment Variables
Each service pulls most configuration from the centralized `config-server`, but local overrides (e.g. DB credentials, GCS credentials) can be supplied via environment variables or a local `application-local.yaml`:

```bash
export DB_URL=jdbc:postgresql://localhost:5432/capstone_db
export DB_USERNAME=postgres
export DB_PASSWORD=your_password
export GCS_BUCKET_NAME=capstone-bucket-v1
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json
export MONGODB_URI=mongodb://localhost:27017/enrollment_db
```

### 3️⃣ Build Each Submodule
```bash
cd student-service && mvn clean install && cd ..
cd program-service && mvn clean install && cd ..
cd enrollment-service && mvn clean install && cd ..
```

### 4️⃣ Run Services Locally
⚠️ Ensure `config-server` (9000) and `service-registry` (9001) from the Platform repo are already running.

```bash
# Terminal 1 — Student Service
cd student-service
mvn spring-boot:run

# Terminal 2 — Program Service
cd program-service
mvn spring-boot:run

# Terminal 3 — Enrollment Service
cd enrollment-service
mvn spring-boot:run
```
Since each service uses a random port, check the console log or the Eureka dashboard (`http://localhost:9001`) to confirm the assigned port and registration status.

## ☁️ Production Deployment (GCP)

Services run on Compute Engine MIGs, managed by PM2, and scale horizontally behind the platform's API Gateway. `student-service` additionally requires the GCE instance's service account to have IAM permissions on the target GCS bucket for profile picture uploads/downloads.

```bash
# Example PM2 startup on a GCE instance
pm2 start "java -jar student-service.jar" --name student-service
pm2 save
```

## 📄 License

This project was developed as part of the Enterprise Cloud Architecture university module (Capstone Project — Alternative Option).
