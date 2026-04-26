# ADA.BI | Analytical Platform System Architecture

**ADA.BI** is a lightweight, high-performance analytical BI platform designed specifically for small businesses to track unit economics and marketing performance. It processes raw data files entirely in-memory using **DuckDB** and persists curated datasets in **Apache Parquet**.

🔗 **[Full Project Case Study & Architecture Docs (Notion Hub)](#вставь_сюда_ссылку_на_твой_Notion)**

## Repository Contents
This repository contains the technical artifacts derived from the System Analysis and Architecture design phases:
* `specs/openapi.yaml` - The complete OpenAPI 3.0 REST contract.
* `db/schema.dbml` - Physical database schema (SQLite + Parquet layout).
* `frontend/charts.html` - ECharts reference configurations for the Dark Glassmorphism UI.

## Tech Stack
* **Data Processing:** DuckDB, Apache Parquet
* **Backend:** Python (Flask)
* **Database:** SQLite (Metadata)
* **Frontend:** Vanilla JS, ECharts, CSS Grid
* **Documentation:** OpenAPI 3.0, DBML, Mermaid.js

---

## Architecture Visualizations

### 1. Data Pipeline (ETL) Sequence
The automated flow from a raw CSV upload to an isolated DuckDB transformation, resulting in a persistent Parquet file.

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant FlaskAPI as Flask API
    participant SQLite
    participant DuckDB
    participant FileSystem

    User->>Frontend: Upload File (CSV/XLSX)
    Frontend->>FlaskAPI: POST /upload (File)
    FlaskAPI->>FileSystem: Save Raw File (uuid.csv)
    FileSystem-->>FlaskAPI: Success
    
    Note over FlaskAPI, FileSystem: Peek first row for headers
    FlaskAPI->>FileSystem: Read Head (Headers)
    FileSystem-->>FlaskAPI: Array of Strings
    FlaskAPI-->>Frontend: Return Columns for Mapping
    
    User->>Frontend: Provide Mapping Config
    Frontend->>FlaskAPI: POST /mapping (JSON)
    FlaskAPI->>SQLite: Save Mapping Config
    FlaskAPI->>DuckDB: Trigger ETL (Extract & Transform)
    DuckDB->>FileSystem: Read Raw File
    DuckDB->>DuckDB: Apply Mapping & Cast Types
    DuckDB->>FileSystem: Save to Parquet (uuid.parquet)
    FileSystem-->>DuckDB: Success
    DuckDB-->>FlaskAPI: ETL Complete
    FlaskAPI->>SQLite: Update processed_file_path
    
    rect rgb(255, 200, 200)
        Note over FlaskAPI, FileSystem: CRITICAL: Finally Block
        FlaskAPI->>FileSystem: Delete Raw File (uuid.csv)
        FileSystem-->>FlaskAPI: Deleted
    end
    
    FlaskAPI-->>Frontend: Redirect to Dashboard
```

### 2. Dynamic Filtering Sequence
Demonstrates how the system instantly recalculates metrics by reading directly from the Parquet file using isolated connections, bypassing full ETL overhead.

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant FlaskAPI as Flask API
    participant SQLite
    participant DuckDBConn as DuckDB (Isolated Conn)
    participant ParquetFile as Parquet File

    User->>Frontend: Select Date Range / Channel
    Frontend->>FlaskAPI: POST /filter (JSON Params)
    FlaskAPI->>SQLite: Get processed_file_path
    SQLite-->>FlaskAPI: Path (uuid.parquet)
    FlaskAPI->>DuckDBConn: Initialize Connection
    DuckDBConn->>ParquetFile: Read Curated Data
    ParquetFile-->>DuckDBConn: Data Stream
    DuckDBConn->>DuckDBConn: Aggregate KPIs (WHERE clause)
    DuckDBConn-->>FlaskAPI: Return Aggregated Row
    FlaskAPI->>FlaskAPI: Format JSON for ECharts
    FlaskAPI-->>Frontend: Return JSON Payload
    Frontend->>User: Update ECharts UI
```
