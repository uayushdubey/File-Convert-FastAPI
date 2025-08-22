CSV to Excel Converter API
Overview

This project provides a FastAPI-based service to convert CSV (or delimited text) files into Excel .xlsx format. It handles large files, detects delimiters automatically, manages multiple sheets for large datasets, and logs row-level errors in a separate sheet. Temporary files are cleaned automatically, ensuring efficient storage management.

Features

Upload CSV files with any delimiter (comma, tab, semicolon, pipe) or auto-detection.

Supports utf-8, latin-1, and utf-16 encoded files.

Automatically splits large CSV files into multiple Excel sheets if row limits exceed Excel’s limit.

Generates an Errors sheet for rows with issues (missing/extra columns).

Automatic temporary file cleanup using a background scheduler.

Cross-Origin Resource Sharing (CORS) support for local and production environments.

Download converted Excel files via a secure endpoint.

Tech Stack

Backend Framework: FastAPI

File Handling & CSV Processing: Python csv, openpyxl

Task Scheduling: APScheduler

Logging: Python logging

Python Version: 3.10+

Storage: Local temporary folder (tmp/)

High-Level Design (HLD)
Components

API Layer (FastAPI)

Handles incoming HTTP requests:

/convert – Upload and convert CSV

/download/{file_name} – Download converted Excel

/ – Health check

File Processor

Reads CSV file.

Detects delimiter and header.

Converts to Excel format.

Manages multi-sheet creation for large files.

Logs errors in a separate sheet.

Temporary Storage

Stores uploaded and converted files temporarily.

Cleans files older than 1 hour using APScheduler.

Error Handling

Catches and logs invalid rows.

Returns structured error responses to clients.

CORS Middleware

Ensures frontend applications from multiple origins can access API endpoints.

HLD Diagram
            +---------------------+
            |  Frontend Client    |
            +---------------------+
                      |
                      v
            +---------------------+
            |      FastAPI        |
            |   Endpoints Layer   |
            +---------------------+
           /          |           \
          v           v            v
 +----------------+ +----------------+ +----------------+
 | CSV Validator  | | Excel Builder  | | Error Logger   |
 +----------------+ +----------------+ +----------------+
           \          |           /
            v         v          v
            +---------------------+
            |   Temporary Storage  |
            |      (tmp/)         |
            +---------------------+
                      |
                      v
            +---------------------+
            | Background Cleanup  |
            +---------------------+

Low-Level Design (LLD)
CSV Conversion Algorithm

Step 1: Upload & Save File

Receive file as UploadFile.

Save to temporary folder (tmp/) with unique UUID name.

Step 2: Detect Delimiter & Header

If delimiter="auto", use csv.Sniffer to detect delimiter.

Determine if file has header row.

Step 3: Read CSV & Populate Excel

Initialize Workbook using openpyxl.

Write headers (bold font).

Iterate through CSV rows:

Validate row length.

Append to current sheet.

If row count exceeds MAX_ROWS_PER_SHEET, create new sheet.

Log any row errors to a list.

Step 4: Write Errors Sheet

If errors exist, create a sheet Errors with columns: Row Number, Raw Row, Error.

Step 5: Adjust Column Widths & Freeze Panes

Auto-fit columns up to 50 characters.

Freeze top row for easy readability.

Step 6: Save Excel & Return URL

Save .xlsx file in tmp/.

Return download URL, detected delimiter, and file size.

Step 7: Cleanup Temporary Files

Upload file deleted after processing.

Downloaded Excel is deleted via background task after sending.

Key Classes/Functions
Function	Purpose
convert_file	Main endpoint to convert CSV to Excel
download_file	Provides Excel file download and schedules cleanup
clean_tmp_folder	Deletes old temporary files every 30 minutes
DELIMITER_MAP	Maps friendly delimiter names to actual symbols
DELIMITER_NAME_MAP	Maps delimiter symbols back to readable names
MAX_ROWS_PER_SHEET	Maximum rows allowed per Excel sheet
Flowchart
Start
  |
  v
[Receive UploadFile]
  |
  v
[Validate File & Encoding]
  |
  v
[Detect Delimiter & Header?] --No--> [Use Default Comma]
  |
  v
[Initialize Workbook]
  |
  v
[Iterate Rows]
  |--> [Check Row Length] --Invalid--> [Add to Errors]
  |
  v
[Append Row to Sheet]
  |
  v
[Max Rows Reached?] --Yes--> [Create New Sheet]
  |
  v
[After All Rows] --> [Add Errors Sheet if needed]
  |
  v
[Adjust Column Width & Freeze Panes]
  |
  v
[Save Excel File]
  |
  v
[Return Download URL & Metadata]
  |
  v
End

API Endpoints
1. Health Check

URL: /

Method: GET

Response:

{
  "message": "Service is running!"
}

2. Convert CSV to Excel

URL: /convert

Method: POST

Form Data:

file (UploadFile) – CSV file

delimiter (string, default "auto") – comma, tab, semicolon, pipe, or auto

encoding (string, default "utf-8") – utf-8, latin-1, utf-16

Response (200 OK):

{
  "download_url": "/download/{file_name}",
  "detected_delimiter": "Comma",
  "file_size": 12345
}


Error Response (400):

{
  "error": "Invalid delimiter selected"
}

3. Download Converted File

URL: /download/{file_name}

Method: GET

Response: Returns .xlsx file and schedules background deletion.

Temporary File Management

All uploaded and converted files are saved in tmp/.

APScheduler deletes files older than 1 hour every 30 minutes.

Downloaded files are removed automatically after serving.

Logging

Logs file cleanup and errors using Python’s logging module.

Example log:

INFO: Deleted old file: 3f9e5c2d.xlsx

Deployment Considerations

CORS: Adjust allow_origins for frontend deployment domain.

Storage: Ensure tmp/ has enough storage for large files.

Scheduler: APScheduler runs in the same process; for production, consider separate job runner for cleanup.

Security: Validate uploaded file types and enforce size limits to prevent abuse.

File Structure
.
├── app.py                  # FastAPI main application
├── tmp/                     # Temporary storage for uploaded and converted files
├── requirements.txt        # Dependencies (FastAPI, openpyxl, apscheduler, uvicorn)
├── README.md               # Project documentation
