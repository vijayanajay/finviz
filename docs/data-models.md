# Data Models for Visual Market Pulse Dashboard

## Overview

This document describes the data structures used in the application, focusing on the EOD stock data fetched from Yahoo Finance and how it's processed and cached.

## Key Data Models

### 1. EOD Stock Data Model
- **Description:** Represents End-of-Day data for a stock or index.
- **Structure (JSON Schema):
  ```json
  {
    "type": "object",
    "properties": {
      "symbol": {"type": "string", "description": "Stock symbol, e.g., 'NIFTY50'"},
      "date": {"type": "string", "format": "date", "description": "Date of the data in YYYY-MM-DD"},
      "open": {"type": "number", "description": "Opening price"},
      "high": {"type": "number", "description": "Highest price of the day"},
      "low": {"type": "number", "description": "Lowest price of the day"},
      "close": {"type": "number", "description": "Closing price"},
      "volume": {"type": "number", "description": "Trading volume"}
    },
    "required": ["symbol", "date", "close"]
  }
  ```
- **Usage:** Fetched via yfinance and cached in the backend.

### 2. Sector Data Model
- **Description:** Aggregates EOD data for stocks within a sector.
- **Structure:
  ```json
  {
    "type": "object",
    "properties": {
      "sector": {"type": "string", "description": "Sector name, e.g., 'finance'"},
      "stocks": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "symbol": {"type": "string"},
            "close": {"type": "number"}
          }
        }
      }
    },
    "required": ["sector", "stocks"]
  }
  ```
- **Usage:** Used in API responses for sector-specific endpoints.

## Caching and Processing
- Data is transformed in the backend before caching to ensure only necessary fields are stored.
- Refer to `docs/architecture.md` for how these models integrate with the system.

For API interactions, see `docs/api-reference.md`.