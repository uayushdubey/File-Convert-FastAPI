# FastAPI CSV to Excel Converter

[![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python-3.7+-blue?style=flat&logo=python)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A high-performance FastAPI service that converts CSV files to Excel format with automatic delimiter detection, encoding support, and intelligent formatting.

## Key Features

- **Smart CSV Detection** - Auto-detects delimiters and encoding
- **Excel Optimization** - Auto-formatting and multi-sheet support  
- **High Performance** - Handles large files with streaming processing
- **Production Ready** - Error handling, logging, and CORS support
- **Auto Cleanup** - Background task management

---

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/uayushdubey/File-Convert-FastAPI.git
cd File-Convert-FastAPI

# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Health Check

```bash
curl http://localhost:8000/
# Response: {"message": "Service is running!"}
```

---

## API Reference

### Core Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Health check |
| `POST` | `/convert` | Convert CSV to Excel |
| `GET` | `/download/{filename}` | Download converted file |

### Convert CSV to Excel

**Endpoint:** `POST /convert`

**Parameters:**

| Parameter | Type | Default | Options |
|-----------|------|---------|---------|
| `file` | File | Required | CSV file |
| `delimiter` | String | `auto` | `auto`, `comma`, `tab`, `semicolon`, `pipe` |
| `encoding` | String | `utf-8` | `utf-8`, `latin-1`, `utf-16` |

**Response:**

```json
{
  "download_url": "/download/filename.xlsx",
  "detected_delimiter": "Comma",
  "file_size": 2048576
}
```

---

## System Architecture

### High-Level Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │───▶│   FastAPI       │───▶│   File System   │
│                 │    │   Service       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   Background    │
                       │   Scheduler     │
                       └─────────────────┘
```

### Data Processing Flow

```
File Upload
    │
    ▼
Validation & Temp Storage
    │
    ▼
Delimiter Detection
    │
    ▼
CSV Parsing (Row-by-Row)
    │
    ▼
Excel Generation
    │
    ├── Sheet 1 (Headers + Data)
    ├── Sheet 2 (If >1M rows)
    └── Error Sheet (If errors)
    │
    ▼
Response with Download URL
    │
    ▼
Background Cleanup
```

### Core Components

**FastAPI Application Server**
- Handles HTTP requests and responses
- Manages file uploads and downloads
- Provides RESTful API endpoints

**CSV Processing Engine**
- Automatic delimiter detection using csv.Sniffer
- Encoding detection and handling
- Row-by-row processing for memory efficiency

**Excel Generation Module**
- OpenPyXL-based Excel file creation
- Multi-sheet support for large datasets
- Auto-formatting and styling

**Background Task Manager**
- APScheduler for periodic cleanup
- Temporary file management
- Error logging and monitoring

---

## Usage Examples

### Python Requests

```python
import requests

# Convert CSV file
files = {'file': open('data.csv', 'rb')}
data = {'delimiter': 'comma', 'encoding': 'utf-8'}
response = requests.post('http://localhost:8000/convert', files=files, data=data)

if response.status_code == 200:
    result = response.json()
    download_url = result['download_url']
    
    # Download converted file
    download_response = requests.get(f'http://localhost:8000{download_url}')
    with open('converted.xlsx', 'wb') as f:
        f.write(download_response.content)
```

### cURL Commands

```bash
# Convert with auto-detection
curl -X POST "http://localhost:8000/convert" \
  -F "file=@sample.csv" \
  -F "delimiter=auto" \
  -F "encoding=utf-8"

# Convert with specific delimiter
curl -X POST "http://localhost:8000/convert" \
  -F "file=@data.csv" \
  -F "delimiter=tab" \
  -F "encoding=latin-1"

# Download converted file
curl -X GET "http://localhost:8000/download/filename.xlsx" \
  --output converted.xlsx
```

### JavaScript Fetch

```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('delimiter', 'auto');
formData.append('encoding', 'utf-8');

fetch('http://localhost:8000/convert', {
    method: 'POST',
    body: formData
})
.then(response => response.json())
.then(data => {
    console.log('Conversion successful:', data);
    window.location.href = `http://localhost:8000${data.download_url}`;
});
```

---

## Error Handling

### HTTP Status Codes

| Code | Description | Response |
|------|-------------|----------|
| `200` | Success | Conversion completed |
| `400` | Bad Request | Invalid parameters or file |
| `404` | Not Found | File not found |
| `500` | Server Error | Processing error |

### Error Response Format

```json
{
  "error": "Invalid delimiter selected"
}
```

### Error Sheet Generation

When CSV parsing errors occur, an additional "Errors" sheet is created:

| Column | Description |
|--------|-------------|
| Row Number | Original CSV row number |
| Raw Row | Complete raw row data |
| Error | Specific error message |

---

## Configuration

### Environment Variables

```bash
# Server Configuration
HOST=0.0.0.0
PORT=8000

# CORS Origins
ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com
```

### Production CORS Setup

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://your-production-domain.com"
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## Performance & Limits

### Specifications

| Metric | Limit |
|--------|-------|
| **Max File Size** | 100MB |
| **Max Rows** | 16 million (multi-sheet) |
| **Concurrent Users** | 1000 connections |
| **Memory Usage** | ~10MB per 1MB CSV |

### Optimization Features

- **Streaming Processing** - Row-by-row parsing prevents memory overflow
- **Background Cleanup** - Automatic file removal every 30 minutes
- **Sheet Splitting** - Automatic handling of Excel row limits (1M+ rows)
- **Column Width Optimization** - Dynamic width calculation

---

## Deployment

### Docker Setup

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Production Server

```bash
# Install production server
pip install gunicorn

# Run with gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        client_max_body_size 100M;
    }
}
```

---

## Testing

### Unit Tests

```python
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_health_check():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Service is running!"}

def test_file_conversion():
    with open("test.csv", "rb") as f:
        response = client.post(
            "/convert",
            files={"file": ("test.csv", f, "text/csv")},
            data={"delimiter": "comma", "encoding": "utf-8"}
        )
    assert response.status_code == 200
    assert "download_url" in response.json()
```

### Load Testing

```bash
# Install Apache Bench
sudo apt-get install apache2-utils

# Test concurrent requests
ab -n 1000 -c 10 http://localhost:8000/
```

---

## Monitoring

### Key Metrics

Monitor these critical metrics:

- **Request Rate** - Requests per second
- **Response Time** - Average processing time
- **Error Rate** - Failed conversion percentage
- **Disk Usage** - Temporary file storage
- **Memory Usage** - Peak memory consumption

### Logging Configuration

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)
```

---

## Contributing

### Development Setup

```bash
# Clone and setup
git clone https://github.com/uayushdubey/File-Convert-FastAPI.git
cd File-Convert-FastAPI
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements-dev.txt

# Run tests
pytest

# Code formatting
black main.py
flake8 main.py
```


**Repository:** [https://github.com/uayushdubey/File-Convert-FastAPI](https://github.com/uayushdubey/File-Convert-FastAPI)
