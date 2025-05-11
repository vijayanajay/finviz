# Data Models for Visual Market Pulse Dashboard

## Overview

This document describes the key data structures used in the application, particularly focusing on the data served by the backend API to the frontend for display. These models are derived from EOD stock data fetched via Yahoo Finance and processed by the backend.

## Key Data Models (for API Responses)

### 1. API Stock Data Model (for Heatmap Constituents)
-   **Description:** Represents the processed stock data for a single constituent within a heatmap (Nifty 50 or a specific sector). This model includes all necessary fields for tile sizing, coloring, and tooltip display as per PRD requirements.
-   **Structure (JSON Schema like):**
    ```json
    {
      "type": "object",
      "properties": {
        "symbol": {"type": "string", "description": "Stock ticker symbol (e.g., RELIANCE.NS)"},
        "name": {"type": "string", "description": "Full stock name (e.g., Reliance Industries Ltd.)"},
        "marketCap": {"type": "number", "description": "Market Capitalization for tile sizing"},
        "eodClosePrice": {"type": "number", "description": "End-of-Day closing price"},
        "eodPercentageChange": {"type": "number", "description": "End-of-Day percentage change from previous close (for tile color)"},
        "eodVolumeChangePercentage": {"type": "number", "description": "End-of-Day volume percentage change from previous day volume (for tooltip)"}
      },
      "required": ["symbol", "name", "marketCap", "eodClosePrice", "eodPercentageChange", "eodVolumeChangePercentage"]
    }
    ```
-   **Usage:** This model is used as an item in the `constituents` array in the API responses for `/api/market/nifty50` and `/api/sector/{sector_identifier}`.

### 2. API Market/Sector Heatmap Data Model
-   **Description:** Represents the overall data structure returned for a full market index (like Nifty 50) or a specific sector heatmap.
-   **Structure (JSON Schema like):**
    ```json
    {
      "type": "object",
      "properties": {
        "dataSource": {"type": "string", "description": "Origin of the data (e.g., Yahoo Finance via yfinance)"},
        "lastUpdated": {"type": "string", "format": "date-time", "description": "ISO 8601 timestamp of last data refresh"},
        "indexName": {"type": "string", "optional": true, "description": "Name of the index, if applicable (e.g., Nifty 50)"},
        "sectorName": {"type": "string", "optional": true, "description": "Name of the sector, if applicable"},
        "sectorIdentifier": {"type": "string", "optional": true, "description": "Identifier of the sector, if applicable"},
        "constituents": {
          "type": "array",
          "items": {"$ref": "#/definitions/ApiStockDataModel"} // Reference to the API Stock Data Model
        }
      },
      "required": ["dataSource", "lastUpdated", "constituents"]
    }
    ```
-   **Usage:** This is the primary model for responses from `/api/market/nifty50` and `/api/sector/{sector_identifier}`.

### 3. API Sector List Item Model
-   **Description:** Represents a single sector in the list of available market sectors.
-   **Structure (JSON Schema like):**
    ```json
    {
      "type": "object",
      "properties": {
        "id": {"type": "string", "description": "Unique identifier for the sector (e.g., nifty_it)"},
        "name": {"type": "string", "description": "Display name of the sector (e.g., Nifty IT)"}
      },
      "required": ["id", "name"]
    }
    ```
-   **Usage:** This model is used as an item in the `sectors` array in the API response for `/api/sectors`.

### 4. API Sector List Data Model
-   **Description:** Represents the overall data structure for the list of available market sectors.
-   **Structure (JSON Schema like):**
    ```json
    {
      "type": "object",
      "properties": {
        "dataSource": {"type": "string", "description": "Origin of the data (e.g., Application Configuration)"},
        "lastUpdated": {"type": "string", "format": "date-time", "description": "ISO 8601 timestamp of last list update/confirmation"},
        "sectors": {
          "type": "array",
          "items": {"$ref": "#/definitions/ApiSectorListItemModel"} // Reference to API Sector List Item Model
        }
      },
      "required": ["dataSource", "lastUpdated", "sectors"]
    }
    ```
-   **Usage:** This is the model for the response from `/api/sectors`.


## Data Transformation and Caching
-   Raw EOD data fetched from `yfinance` (which might include open, high, low, close, volume, etc.) is transformed by the backend's "Business Logic" component.
-   This transformation includes:
    -   Calculating `eodPercentageChange` based on the current and previous day's close prices.
    -   Calculating `eodVolumeChangePercentage` based on the current and previous day's volume.
    -   Mapping raw data to the `API Stock Data Model` to ensure only necessary fields are sent to the frontend, optimizing payload size.
-   The transformed data, ready for API responses, is then cached by the `Caching Service` to improve performance and reduce load on `yfinance`.

Refer to `docs/architecture.md` for how these models integrate with the system components and `docs/api-reference.md` for their usage in API endpoints.