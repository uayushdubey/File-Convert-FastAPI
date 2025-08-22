# FastAPI CSV to Excel Converter

A high-performance FastAPI service that converts CSV files to Excel format with automatic delimiter detection, encoding support, and intelligent formatting.

## Features

- **Automatic Delimiter Detection**: Smart detection of CSV delimiters (comma, tab, semicolon, pipe)
- **Multiple Encoding Support**: UTF-8, Latin-1, UTF-16 encoding options
- **Large File Handling**: Automatic splitting across multiple Excel sheets for large datasets
- **Error Handling**: Comprehensive error tracking with dedicated error sheets
- **Auto-formatting**: Column width optimization and header styling
- **Background Cleanup**: Automatic temporary file cleanup
- **CORS Support**: Ready for frontend integration
- **Production Ready**: Includes logging, scheduling, and proper error handling

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [API Endpoints](#api-endpoints)
- [Configuration](#configuration)
- [High-Level Design (HLD)](#high-level-design-hld)
- [Low-Level Design (LLD)](#low-level-design-lld)
- [Usage Examples](#usage-examples)
- [Error Handling](#error-handling)
- [Performance Considerations](#performance-considerations)
- [Deployment](#deployment)
- [Contributing](#contributing)

## Installation

### Prerequisites

- Python 3.7+
- pip package manager

### Dependencies

```bash
pip install fastapi uvicorn openpyxl apscheduler python-multipart
```

### Setup

```bash
git clone https://github.com/uayushdubey/File-Convert-FastAPI.git
cd File-Convert-FastAPI
pip install -r requirements.txt
```

## Quick Start

### Run the Server

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Health Check

```bash
curl http://localhost:8000/
```

Response:
```json
{
  "message": "Service is running!"
}
```

## API Endpoints

### 1. Health Check
- **Endpoint**: `GET /`
- **Description**: Service health status
- **Response**: `{"message": "Service is running!"}`

### 2. Convert CSV to Excel
- **Endpoint**: `POST /convert`
- **Content-Type**: `multipart/form-data`

#### Request Parameters

| Parameter | Type | Default | Options | Description |
|-----------|------|---------|---------|-------------|
| file | File | Required | - | CSV file to convert |
| delimiter | String | "auto" | "auto", "comma", "tab", "semicolon", "pipe" | CSV delimiter type |
| encoding | String | "utf-8" | "utf-8", "latin-1", "utf-16" | File encoding |

#### Response

```json
{
  "download_url": "/download/filename.xlsx",
  "detected_delimiter": "Comma",
  "file_size": 2048576
}
```

### 3. Download Converted File
- **Endpoint**: `GET /download/{file_name}`
- **Description**: Download the converted Excel file
- **Response**: Excel file with automatic cleanup

## Configuration

### Environment Variables

```bash
# Server Configuration
HOST=0.0.0.0
PORT=8000

# CORS Origins (modify for production)
ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com
```

### CORS Configuration

Update the `allow_origins` list in the code for production:

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

## High-Level Design (HLD)

### System Architecture

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

### Core Components

1. **FastAPI Application Server**
   - Handles HTTP requests and responses
   - Manages file uploads and downloads
   - Provides RESTful API endpoints

2. **CSV Processing Engine**
   - Automatic delimiter detection using csv.Sniffer
   - Encoding detection and handling
   - Row-by-row processing for memory efficiency

3. **Excel Generation Module**
   - OpenPyXL-based Excel file creation
   - Multi-sheet support for large datasets
   - Auto-formatting and styling

4. **Background Task Manager**
   - APScheduler for periodic cleanup
   - Temporary file management
   - Error logging and monitoring

5. **File Management System**
   - Temporary file storage
   - Automatic cleanup mechanisms
   - UUID-based file naming

### Data Flow

1. **File Upload**: Client uploads CSV file with optional parameters
2. **Validation**: Server validates file format and parameters
3. **Processing**: CSV parsing with delimiter detection and encoding handling
4. **Conversion**: Row-by-row Excel generation with formatting
5. **Response**: Return download URL and metadata
6. **Download**: Client downloads converted file
7. **Cleanup**: Background task removes temporary files

## Low-Level Design (LLD)

### Class Structure

```python
# Core Application Structure
FastAPI App
├── Middleware (CORS)
├── Routes
│   ├── GET /
│   ├── POST /convert
│   └── GET /download/{file_name}
├── Background Tasks
│   └── File Cleanup Scheduler
└── Utility Functions
    ├── CSV Processing
    ├── Excel Generation
    └── File Management
```

### Key Functions

#### 1. convert_file()
```python
async def convert_file(file: UploadFile, delimiter: str, encoding: str)
```
- **Input**: CSV file, delimiter preference, encoding type
- **Process**: 
  - File validation and temporary storage
  - Delimiter detection using csv.Sniffer
  - Row-by-row CSV parsing
  - Excel workbook creation with multiple sheets
  - Error tracking and formatting
- **Output**: Download URL, detected delimiter, file size

#### 2. clean_tmp_folder()
```python
def clean_tmp_folder()
```
- **Purpose**: Remove files older than 1 hour
- **Schedule**: Every 30 minutes
- **Logic**: Compare file modification time with current time

#### 3. Excel Processing Logic
```python
# Multi-sheet handling
if sheet_row_count >= MAX_ROWS_PER_SHEET:
    sheet_num += 1
    sheet = wb.create_sheet(f"Sheet_{sheet_num}")
    # Reset row counter and add headers
```

### Data Structures

#### Delimiter Mapping
```python
DELIMITER_MAP = {
    "comma": ",",
    "tab": "\t", 
    "semicolon": ";",
    "pipe": "|"
}
```

#### Error Tracking
```python
error_rows = [(row_number, raw_row_data, error_message)]
```

### Memory Management

- **Streaming Processing**: Files processed row-by-row to handle large datasets
- **Temporary File Cleanup**: Automatic removal prevents disk space issues
- **Sheet Splitting**: Large datasets split across multiple Excel sheets (1M+ rows)
- **Column Width Optimization**: Dynamic width calculation with 50-character limit

### Security Considerations

- **File Type Validation**: Only CSV files accepted
- **Encoding Safety**: Error handling with 'replace' mode
- **Path Security**: UUID-based temporary file naming
- **CORS Protection**: Configurable origin restrictions
- **Input Sanitization**: Delimiter and encoding validation

## Usage Examples

### Python Requests

```python
import requests

# Basic conversion
files = {'file': open('data.csv', 'rb')}
data = {'delimiter': 'comma', 'encoding': 'utf-8'}
response = requests.post('http://localhost:8000/convert', files=files, data=data)

if response.status_code == 200:
    result = response.json()
    download_url = result['download_url']
    
    # Download the converted file
    download_response = requests.get(f'http://localhost:8000{download_url}')
    with open('converted.xlsx', 'wb') as f:
        f.write(download_response.content)
```

### cURL Examples

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
// File upload and conversion
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
    // Download the file
    window.location.href = `http://localhost:8000${data.download_url}`;
});
```

## Error Handling

### Client Errors (400)

```json
{
  "error": "Invalid delimiter selected"
}
```

```json
{
  "error": "Invalid encoding selected"  
}
```

```json
{
  "error": "Empty file"
}
```

### File Not Found (404)

```json
{
  "detail": "File not found"
}
```

### Error Sheet Generation

When CSV parsing errors occur, an additional "Errors" sheet is created containing:

| Column | Description |
|--------|-------------|
| Row Number | Original CSV row number |
| Raw Row | Complete raw row data |
| Error | Specific error message |

## Performance Considerations

### Scalability Limits

- **File Size**: Recommended maximum 100MB per file
- **Row Count**: Up to 16 million rows (across multiple sheets)
- **Concurrent Users**: Default FastAPI handles ~1000 concurrent connections
- **Memory Usage**: ~10MB RAM per 1MB CSV file during processing

### Optimization Features

- **Streaming Processing**: Row-by-row parsing prevents memory overflow
- **Background Cleanup**: Prevents disk space accumulation
- **Column Width Caching**: Efficient width calculation
- **Sheet Splitting**: Automatic handling of Excel row limits

### Performance Monitoring

```python
# Add timing middleware for monitoring
import time

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

## Deployment

### Docker Deployment

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Production Configuration

```bash
# Install production server
pip install gunicorn

# Run with gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

### Environment Setup

```bash
# production.env
PYTHONPATH=/app
LOG_LEVEL=INFO
MAX_WORKERS=4
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

## Monitoring and Logging

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

### Health Metrics

Monitor these key metrics:

- **Request Rate**: Requests per second
- **Response Time**: Average processing time
- **Error Rate**: Failed conversion percentage
- **Disk Usage**: Temporary file storage
- **Memory Usage**: Peak memory consumption

## Contributing

### Development Setup

```bash
# Clone repository
git clone https://github.com/uayushdubey/File-Convert-FastAPI.git
cd File-Convert-FastAPI

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -r requirements-dev.txt

# Run tests
pytest

# Code formatting
black main.py
flake8 main.py
```

### Contribution Guidelines

1. **Fork the repository** and create a feature branch
2. **Write tests** for new functionality
3. **Follow PEP 8** coding standards
4. **Update documentation** for API changes
5. **Submit pull request** with detailed description

### Code Standards

- **Type Hints**: Use type annotations for all functions
- **Docstrings**: Document all public functions
- **Error Handling**: Comprehensive exception handling
- **Testing**: Minimum 80% code coverage

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

### Version 1.0.0
- Initial release
- Basic CSV to Excel conversion
- Automatic delimiter detection
- Multi-encoding support
- Background file cleanup
- CORS support
- Production-ready logging

---

**Repository**: [https://github.com/uayushdubey/File-Convert-FastAPI](https://github.com/uayushdubey/File-Convert-FastAPI)
