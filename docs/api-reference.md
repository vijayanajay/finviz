# API Reference for Visual Market Pulse Dashboard

## Overview

This document outlines the RESTful APIs exposed by the Flask backend. These endpoints handle data fetching, caching, and delivery to the frontend, focusing on EOD stock data. All API endpoints are designed to provide the necessary data for frontend rendering as specified in the PRD.

## Base URL

All API endpoints are prefixed with `/api`. For example: `/api/market/nifty50`.

## Common Response Elements

-   **`dataSource`**: Indicates the origin of the data (e.g., "Yahoo Finance via yfinance").
-   **`lastUpdated`**: ISO 8601 timestamp indicating when the EOD data was last fetched/updated by the backend.

## API Endpoints

### 1. Get Nifty 50 EOD Data
-   **Endpoint:** `/api/market/nifty50`
-   **Method:** GET
-   **Description:** Fetches End-of-Day (EOD) data for all constituent stocks of the Nifty 50 index. Data includes details required for heatmap visualization (market cap, percentage change) and tooltips (name, symbol, EOD price, EOD % change, EOD volume % change).
-   **Query Parameters:** None for MVP. (Future: `date` for historical data).
-   **Response (200 OK):**
    ```json
    {
      "dataSource": "Yahoo Finance via yfinance",
      "lastUpdated": "2025-05-11T18:00:00Z",
      "indexName": "Nifty 50",
      "constituents": [
        {
          "symbol": "RELIANCE.NS",
          "name": "Reliance Industries Ltd.",
          "marketCap": 15000000000000,
          "eodClosePrice": 2750.50,
          "eodPercentageChange": 1.25,
          "eodVolumeChangePercentage": 15.5
        },
        {
          "symbol": "TCS.NS",
          "name": "Tata Consultancy Services Ltd.",
          "marketCap": 12000000000000,
          "eodClosePrice": 3200.75,
          "eodPercentageChange": -0.50,
          "eodVolumeChangePercentage": -5.2
        }
        // ... other Nifty 50 stocks
      ]
    }
    ```
-   **Caching:** Backend data is cached, typically refreshing once daily after market close.

### 2. Get List of Supported Market Sectors
-   **Endpoint:** `/api/sectors`
-   **Method:** GET
-   **Description:** Fetches a list of all available market sectors that the user can select for heatmap visualization. Each sector has an identifier used to fetch its specific data.
-   **Query Parameters:** None.
-   **Response (200 OK):**
    ```json
    {
      "dataSource": "Application Configuration", // Or derived from backend logic
      "lastUpdated": "2025-05-11T18:00:00Z", // Timestamp of when this list was last confirmed/updated
      "sectors": [
        {"id": "nifty_bank", "name": "Nifty Bank"},
        {"id": "nifty_it", "name": "Nifty IT"},
        {"id": "nifty_auto", "name": "Nifty Auto"},
        {"id": "nifty_fmcg", "name": "Nifty FMCG"},
        {"id": "nifty_pharma", "name": "Nifty Pharma"}
        // ... other supported sectors
      ]
    }
    ```
-   **Caching:** This list is relatively static and can be cached aggressively by the backend and/or frontend.

### 3. Get EOD Data for a Specific Market Sector
-   **Endpoint:** `/api/sector/{sector_identifier}`
-   **Method:** GET
-   **Description:** Fetches End-of-Day (EOD) data for all constituent stocks of a specific market sector, identified by `{sector_identifier}`. Data structure is similar to the Nifty 50 endpoint.
-   **Path Parameters:**
    -   `sector_identifier` (string, required): The unique identifier for the sector (e.g., "nifty_it").
-   **Query Parameters:** None for MVP.
-   **Response (200 OK):**
    ```json
    {
      "dataSource": "Yahoo Finance via yfinance",
      "lastUpdated": "2025-05-11T18:00:00Z",
      "sectorName": "Nifty IT", // Name corresponding to the identifier
      "sectorIdentifier": "nifty_it",
      "constituents": [
        {
          "symbol": "INFY.NS",
          "name": "Infosys Ltd.",
          "marketCap": 7000000000000,
          "eodClosePrice": 1500.00,
          "eodPercentageChange": 0.75,
          "eodVolumeChangePercentage": 10.1
        },
        {
          "symbol": "WIPRO.NS",
          "name": "Wipro Ltd.",
          "marketCap": 3000000000000,
          "eodClosePrice": 450.20,
          "eodPercentageChange": -1.10,
          "eodVolumeChangePercentage": 2.3
        }
        // ... other stocks in this sector
      ]
    }
    ```
-   **Response (404 Not Found):** If the `sector_identifier` is invalid.
    ```json
    {
      "error": "Sector not found",
      "message": "The requested sector identifier 'invalid_sector_id' does not exist."
    }
    ```
-   **Caching:** Backend data is cached, typically refreshing once daily.

### 4. Get Metadata (Optional - As per PRD Section 7)
-   **Endpoint:** `/api/metadata`
-   **Method:** GET
-   **Description:** Provides metadata about the EOD data, such as the last update timestamp. This information might also be included in other API responses.
-   **Response (200 OK):**
    ```json
    {
      "eodDataLastUpdated": "2025-05-11T18:00:00Z",
      "dataSource": "Yahoo Finance via yfinance"
    }
    ```

## Authentication

No authentication is required for these public APIs in the MVP, as per PRD.

## Error Responses

Generic error responses will follow a consistent structure.

-   **400 Bad Request:** Invalid parameters or request format.
    ```json
    {
      "error": "Bad Request",
      "message": "Details about the validation error."
    }
    ```
-   **404 Not Found:** Requested resource does not exist (e.g., invalid sector ID).
    ```json
    {
      "error": "Not Found",
      "message": "The requested resource was not found."
    }
    ```
-   **500 Internal Server Error:** An unexpected error occurred on the server (e.g., issues fetching data from `yfinance` that couldn't be handled gracefully).
    ```json
    {
      "error": "Internal Server Error",
      "message": "An unexpected error occurred. Please try again later."
    }
    ```

For implementation details, refer to `docs/architecture.md` and backend code in `src/backend/`.