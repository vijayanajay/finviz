# Visual Market Pulse Dashboard Architecture Document

## Technical Summary

The Visual Market Pulse Dashboard is a web application designed to provide a quick, visualization-driven overview of the Indian stock market, focusing on End-of-Day (EOD) data for the Nifty 50 index and key market sectors. The architecture employs a client-server model. The frontend is built with Alpine.js and Bootstrap 5 for a responsive and interactive user experience. The backend is a Flask (Python) application responsible for serving the frontend, fetching EOD data from Yahoo Finance (via the `yfinance` library), caching this data, and exposing it through a set of RESTful APIs. The system is designed for deployment on Render, utilizing Docker for containerization. Key architectural patterns include RESTful APIs, server-side caching, and component-based UI.

## High-Level Overview

The system follows a traditional three-tier architecture: a presentation tier (frontend), an application tier (backend API and logic), and an external data source tier (Yahoo Finance). Users interact with the frontend, which makes requests to the backend API. The backend, in turn, fetches data from Yahoo Finance, processes it, caches it, and returns it to the frontend for display. The primary focus is on efficient EOD data delivery and intuitive visualization.

```mermaid
graph TD
    User[End User] -->|Interacts via Browser| FE[Frontend (Alpine.js, Bootstrap 5 on Render Static Hosting)]
    FE -->|API Calls (HTTPS)| BE[Backend API (Flask, Python on Render)]
    BE -->|Data Fetching & Caching| Cache[(Flask-Caching: Redis/File)]
    BE -->|Uses yfinance library| YFIN[Yahoo Finance (External Data Source)]
    Cache --> BE
```

## Component View

The system comprises the following major logical components:

```mermaid
graph TD
    subgraph Frontend (Client-Side)
        direction LR
        UI_Views[UI Views/Pages (HTML, Bootstrap)]
        Alpine_Components[Alpine.js Components (Heatmap, Tooltip, SectorSelector)]
        State_Mgmt[Client-Side State (Alpine.js Store & x-data)]
        API_Client[API Client (fetch API)]
    end

    subgraph Backend (Server-Side on Render)
        direction LR
        API_Endpoints[API Endpoints (Flask Blueprints)]
        Business_Logic[Business Logic (Data Transformation, Edge Case Handling)]
        YFinance_Service[yfinance Service (Data Fetching)]
        Caching_Service[Caching Service (Flask-Caching)]
    end

    External_Services[External Services]

    User[User] --> UI_Views
    UI_Views --> Alpine_Components
    Alpine_Components --> State_Mgmt
    Alpine_Components --> API_Client
    API_Client --> API_Endpoints
    API_Endpoints --> Business_Logic
    Business_Logic --> YFinance_Service
    Business_Logic --> Caching_Service
    YFinance_Service --> External_Services
    Caching_Service -.->|Stores/Retrieves| Cached_Data[Cached EOD Data]

    subgraph External_Services
        YFinance[Yahoo Finance API]
    end
```

  **Frontend (Alpine.js & Bootstrap 5):**
    -   **UI Views/Pages:** Renders the HTML structure and overall layout using Bootstrap 5.
    -   **Alpine.js Components:** Manages interactive UI elements like the heatmap grid, stock tiles, tooltips, and sector selector. Handles dynamic updates based on user interaction and data fetched from the backend. Detailed component structure will be elaborated in `docs/frontend-architecture.md`.
    -   **Client-Side State Management:** Uses Alpine.js's `x-data` for component-level state and `$store` for shared global state (e.g., current view, loading/error status).
    -   **API Client:** Responsible for making HTTP requests to the backend API and handling responses.
-   **Backend (Flask, Python):**
    -   **API Endpoints (Flask Blueprints):** Exposes RESTful APIs for market data, sector lists, and sector-specific data as defined in `docs/api-reference.md`. API responses are structured to provide all data needed by the frontend as per PRD.
    -   **Business Logic:** Handles data transformation from raw `yfinance` output to the defined API Stock Data Model (see `docs/data-models.md`), including calculation of percentage changes and preparation for API responses. It also manages scenarios like missing data for individual stocks from `yfinance`, ensuring the API provides a consistent structure (e.g., with null fields or an error indicator for that specific stock) to prevent frontend errors.
    -   **yfinance Service:** Encapsulates logic for fetching EOD stock data using the `yfinance` library.
    -   **Caching Service (Flask-Caching):** Implements server-side caching (Redis preferred, fallback to file-based on Render) to store EOD data, reducing redundant calls to `yfinance` and improving response times. Cache updates daily post-market close.
-   **External Services:**
    -   **Yahoo Finance:** The sole source for EOD stock market data, accessed via the `yfinance` library.

(The `src/` directory structure is detailed in `docs/project-structure.md`)

## Key Architectural Decisions & Patterns

-   **Technology Stack:** Python/Flask for the backend, Alpine.js/Bootstrap 5 for the frontend. Specific versions are detailed in `docs/tech-stack.md`. (Rationale: PRD specified, lightweight and suitable for MVP). While the PRD was prescriptive, alternatives like React/Vue for frontend or Django/FastAPI for backend were implicitly considered. The chosen stack was favored for its lightweight nature and rapid development for MVP.
-   **API Design:** RESTful APIs for clear separation between frontend and backend. (Rationale: Industry standard, well-understood).
-   **Data Sourcing:** Exclusively EOD data from `yfinance`. (Rationale: PRD requirement, simplifies MVP).
-   **Caching:** Server-side caching of `yfinance` data is critical for performance and to respect potential rate limits. Daily refresh. (Rationale: PRD NFR-07, performance).
-   **Deployment:** Containerized deployment on Render using Docker, Gunicorn, and Nginx. (Rationale: PRD specified, modern deployment practice).
-   **Modularity:** Flask Blueprints for backend API organization; Component-based UI with Alpine.js. (Rationale: Maintainability, scalability).
-   **Error Handling:** Graceful error handling in backend (consistent API error responses) and frontend (user-friendly messages). The Business Logic component will also handle specific data-related edge cases. (Rationale: PRD NFR-07).
-   **State Management (Frontend):** Alpine.js for lightweight, localized state and simple global stores. (Rationale: Sufficient for MVP complexity, avoids heavier frameworks).
-   **Security Considerations:** Security measures outlined in PRD NFR-06 will be addressed: basic input sanitization will be handled within Flask request processing; rate limiting can be configured at the Nginx/reverse proxy level on Render; and Content Security Policy (CSP) headers will be set by the Flask application. Environment variables will be used for any sensitive configuration, avoiding exposure in code. HTTPS is enforced for all API calls.
-   **Operational Considerations:** Logging will be implemented using Python's standard logging module within the Flask application, configured to output structured logs. On Render, these logs can be aggregated and monitored through the platform's logging services. Health check endpoints will be provided for the Flask application.
-   **Accessibility:** The choice of Bootstrap 5 aids in adhering to WCAG 2.1 Level AA guidelines for accessibility, which will be a focus during frontend development.

(See `docs/coding-standards.md` for detailed coding patterns and error handling)

## Infrastructure and Deployment Overview

-   **Cloud Provider(s):** Render.
-   **Core Services Used (Render):** Web Service (for Flask app), Static Site (for frontend assets), Redis (if chosen for caching).
-   **Infrastructure as Code (IaC):** Potentially `render.yaml` for service configuration. Manual setup for MVP is also possible on Render.
-   **Deployment Strategy:** CI/CD pipeline (e.g., GitHub Actions) deploying to Render.
    -   Backend: Build Docker image, push to registry, deploy on Render Web Service.
    -   Frontend: Build static assets (if any build step is introduced later, otherwise direct deployment), deploy to Render Static Site.
-   **Environments:**
    -   `development` (local)
    -   `production` (on Render)
    -   (Optional: `staging` on Render)

(See `docs/environment-vars.md` for configuration details)

## Key Reference Documents

-   `docs/prd.md`
-   `docs/tech-stack.md`
-   `docs/project-structure.md`
-   `docs/coding-standards.md`
-   `docs/api-reference.md`
-   `docs/data-models.md`
-   `docs/environment-vars.md`
-   `docs/testing-strategy.md`
-   `docs/frontend-architecture.md` (To be detailed further)

## Change Log

| Change                 | Date       | Version | Description                                                                                                                               | Author                     |
| ---------------------- | ---------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| Initial draft          | 2025-05-11 | 0.1     | Initial draft based on PRD                                                                                                                | Gemini 2.5 Pro (Architect Agent) |
| Review & Enhancements  | 2025-05-11 | 0.2     | Added details on error handling for missing data, security controls, logging, accessibility note, and clarified API/data model references. | Gemini 2.5 Pro (Architect Agent) |
