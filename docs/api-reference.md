# API Reference for Visual Market Pulse Dashboard

## Overview

This document outlines the RESTful APIs exposed by the Flask backend. These endpoints handle data fetching, caching, and delivery to the frontend, focusing on EOD stock data.

## API Endpoints

All endpoints are prefixed with `/api/v1/` and return JSON responses. They use HTTP methods (GET, POST) and include error handling for invalid requests.

### 1. Get Nifty 50 EOD Data
- **Endpoint:** `/api/v1/nifty50`
- **Method:** GET
- **Description:** Fetches End-of-Day data for the Nifty 50 index.
- **Query Parameters:**
  - `date`: Optional, YYYY-MM-DD format for specific date (defaults to latest).
- **Response:**
  ```json
  {
    "symbol": "NIFTY50",
    "data": [
      {"date": "2023-10-01", "close": 19500.50, "open": 19400.00}
    ]
  }
  ```
- **Caching:** Data is cached for 24 hours.

### 2. Get Sector Data
- **Endpoint:** `/api/v1/sectors/{sector}`
- **Method:** GET
- **Description:** Retrieves EOD data for a specific sector (e.g., 'finance', 'tech').
- **Path Parameters:**
  - `sector`: String, e.g., 'finance'.
- **Query Parameters:**
  - `limit`: Optional, number of stocks to return (default: 10).
- **Response Example:**
  ```json
  {
    "sector": "finance",
    "stocks": [
      {"symbol": "HDFCBANK", "close": 1500.25}
    ]
  }
  ```

### 3. Search Stocks
- **Endpoint:** `/api/v1/search`
- **Method:** GET
- **Description:** Searches for stocks by name or symbol.
- **Query Parameters:**
  - `query`: Required, search term.
- **Response:** List of matching stock symbols and basic data.

## Authentication

No authentication required for these public APIs in the MVP. (Future: Consider API keys for production.)

## Error Responses

- **400 Bad Request:** Invalid parameters.
- **500 Internal Server Error:** Data fetching issues.

For implementation details, refer to `docs/architecture.md` and backend code in `src/backend/`.