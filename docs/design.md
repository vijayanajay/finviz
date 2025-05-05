### Architecture Design: Visual Market Pulse Dashboard - v1.0

**Version:** 1.0
**Date:** 2025-05-05
**Author:** Gemini 2.5 Pro (acting as Senior Software Engineer)

#### 1. Overview

This document details the software architecture for the Visual Market Pulse Dashboard frontend and its supporting backend API. The primary goal is to deliver a highly scannable, visualization-driven overview of the Indian stock market (Nifty 50/Next 50) using daily or greater granularity data sourced exclusively from `yfinance` via a backend API.

The design prioritizes:

*   **Correctness:** Accurately reflecting the requirements defined in PRD v1.2.
*   **Simplicity:** Employing straightforward patterns, minimal abstractions, and leveraging standard library/framework features.
*   **Maintainability:** Ensuring code is easy to understand, modify, and debug through clear separation of concerns and adherence to conventions.
*   **Requirement Alignment:** Directly mapping architectural components to PRD features (FEAT-xxx, FR-xxx, NFR-xxx).
*   **Clear Frontend/Backend Separation:** Strictly defining responsibilities between the client-side browser execution environment and the server-side API/rendering logic.

#### 2. Guiding Principles

*   **Backend is API-First:** The Flask backend primarily serves data (JSON) or pre-rendered HTML fragments (for HTMX) via well-defined API endpoints. It does *not* handle complex frontend state management.
*   **Thin Client (Browser):** The browser renders HTML provided by the backend, uses Bootstrap for layout/styling, executes minimal JavaScript (primarily for chart initialization/updates), and leverages HTMX for targeted dynamic interactions.
*   **Leverage Frameworks:** Utilize Bootstrap for responsive UI and Flask for routing, request handling, and templating, minimizing bespoke solutions.
*   **Stateless Backend Services:** API endpoints should be stateless wherever possible, relying on request parameters for context. Caching is used for performance, not for maintaining user session state.
*   **Explicit Data Flow:** Data fetching and processing logic resides solely in the backend; the frontend consumes data from the API.
*   **Configuration over Code:** Define parameters like index lists, color scales (once defined), and API endpoints in configuration files or environment variables where practical.

#### 3. High-Level Architecture

The system consists of two main parts:

1.  **Frontend (Client-Side):** Runs in the user's browser. Responsible for rendering the UI (HTML/CSS via Bootstrap), handling user interactions, initiating requests (via standard navigation, HTMX, or minimal JS), and displaying data visualizations (D3.js).
2.  **Backend (Server-Side):** A Flask application responsible for:
    *   Serving HTML pages (potentially pre-rendered with initial data).
    *   Providing API endpoints (`/api/...`) to deliver market data (JSON) or HTML fragments.
    *   Fetching data exclusively from `yfinance` (using the `yfinance` library).
    *   Performing necessary calculations (e.g., daily % change, volume % change).
    *   Handling request routing and basic business logic.
    *   Implementing caching for performance.

```mermaid
graph LR
    subgraph User Browser (Client-Side)
        direction LR
        UserInteraction[User Interaction (Click, Hover, Tap)] --> BrowserUI[HTML/CSS/JS <br> (Bootstrap, HTMX, D3.js)];
        BrowserUI -- HTTP Request --> FlaskBE;
        BrowserUI -- HTMX Request --> FlaskBE;
        BrowserUI -- API Request (JS) --> FlaskBE;
    end

    subgraph Server-Side
        direction TB
        FlaskBE[Flask Backend App];
        FlaskBE -- Serves HTML/Fragments --> BrowserUI;
        FlaskBE -- Serves JSON --> BrowserUI;

        subgraph Flask Components
            direction TB
            FlaskRoutes[Routes/Views <br> (/, /stock/*, /index/*)] --> FlaskAPI[API Endpoints <br> (/api/...)];
            FlaskAPI --> DataService[Data Fetching Service];
            FlaskRoutes --> TemplateEngine[Template Engine <br> (Jinja2)];
        end

        DataService --> Cache[(Cache <br> e.g., Flask-Caching/Redis)];
        DataService -- yfinance lib --> YFinanceAPI[Yahoo Finance (yfinance)];
        Cache -- If Cache Miss --> DataService;
        FlaskBE --> FlaskComponents;
    end

    style UserBrowser fill:#f9f,stroke:#333,stroke-width:2px;
    style ServerSide fill:#ccf,stroke:#333,stroke-width:2px;
```

#### 4. Technology Stack

*   **Backend:**
    *   **Framework:** Flask (Python) - Lightweight, simple, suitable for APIs and template rendering.
    *   **Data Source Library:** `yfinance` - For fetching stock/index data.
    *   **WSGI Server:** Gunicorn / Waitress (for production deployment).
    *   **Caching (Optional but Recommended):** Flask-Caching with a suitable backend (e.g., SimpleCache, Redis).
*   **Frontend:**
    *   **Structure/Layout:** HTML5.
    *   **Styling:** Bootstrap 5 - For responsive design, grid system, and pre-built components. Minimal custom CSS.
    *   **Dynamic Interactions:** HTMX - For partial page updates (tooltips, potentially dynamic data loading) triggered via HTML attributes.
    *   **Charting:** D3.js (as per PRD) - For rendering the neutral-colored line charts. Requires minimal JavaScript for initialization and data updates. *Note: Consider if a simpler charting library like Chart.js could meet requirements with less complexity, but proceeding with D3.js as specified.*
    *   **Core Logic:** Minimal Vanilla JavaScript - Primarily for initializing D3 charts and potentially handling chart timeframe selection logic if not done via HTMX.

#### 5. Component Breakdown

##### 5.1. Backend (Flask Application)

*   **`app.py` / Main Application:** Initializes Flask app, configures extensions (Caching), registers Blueprints.
*   **`routes.py` / `views.py` (or Blueprints):**
    *   Defines core page routes:
        *   `@app.route('/')`: Renders the homepage template (`index.html`). May include initial Nifty 50 heatmap data or trigger an HTMX load.
        *   `@app.route('/stock/<symbol>')`: Renders the stock detail page template (`stock_detail.html`), passing the stock symbol.
        *   `@app.route('/index/<index_name>')` (Release 2): Renders the index detail page template (`index_detail.html`), passing the index name.
*   **`api.py` (or API Blueprint):**
    *   Defines API endpoints, all returning JSON or HTML fragments:
        *   `GET /api/heatmap/{index_name}`: Fetches data via `data_service`, calculates daily % change for all stocks in the index, returns JSON `[{'symbol': 'XYZ', 'change': 1.5}, ...]`. (Covers FR-1.1, FR-1.2, FR-1.4).
        *   `GET /api/tooltip/{symbol}`: Fetches data via `data_service`, calculates daily % change and volume change, returns an HTML fragment containing the required info (Stock Name, % Change, Volume Change) styled appropriately for a tooltip. (Covers FEAT-002, FR-2.3). Triggered by HTMX.
        *   `GET /api/stock/{symbol}/details`: Fetches current price info (LTP, daily change) via `data_service`, returns JSON `{'name': 'Stock Name', 'symbol': 'XYZ', 'price': 100.0, 'change': 1.5}`. (Covers FR-3.1, FR-3.2).
        *   `GET /api/stock/{symbol}/chart?timeframe=<tf>`: Fetches historical data (1D+ granularity) for the given symbol and timeframe (`1D`, `5D`, `1M`, `6M`, `1Y`, `Max` - definition TBD) via `data_service`. Returns JSON `{'timestamps': [...], 'prices': [...]}`. (Covers FEAT-004, FR-4.4, FR-5.2).
        *   `GET /api/index/{index_name}/top_movers` (Release 2): Fetches data for the index via `data_service`, calculates daily % change, determines Top 5 Gainers/Losers (handling ties - logic TBD), returns JSON `{'gainers': [...], 'losers': [...]}`. (Covers FR-7.2, FEAT-009, FR-9.1, FR-9.2, FR-9.3).
        *   `GET /api/index/{index_name}/summary` (Release 3): Fetches overall index daily % change via `data_service`, returns JSON `{'index': 'Nifty 50', 'change': 0.5}`. (Covers FEAT-014, FR-14.2).
*   **`data_service.py`:**
    *   Contains functions to interact with the `yfinance` library.
    *   Handles fetching historical data (`history()`) for specified tickers and periods/intervals (min '1d').
    *   Handles fetching stock info (`Ticker().info`).
    *   Implements caching logic (using `@cache.memoize` or similar) to avoid redundant `yfinance` calls.
    *   Includes logic for calculating Daily % Change and Daily Volume Change based on closing prices and volumes. **Requires clear definition of "Current Day" vs "Previous Day" based on data availability timing (TBD).**
    *   Includes error handling for `yfinance` API calls (e.g., invalid ticker, network issues).
*   **`models.py` (Optional):** Simple data classes or dictionaries for structuring data returned by `data_service`.
*   **`config.py`:** Stores configuration like Nifty 50/Next 50 stock lists, cache settings, potentially the color scale mapping (once defined).
*   **`templates/`:** Jinja2 HTML templates.
    *   `base.html`: Base template with Bootstrap CSS/JS includes, navigation structure (minimal), footer (for attribution/disclaimers - placement TBD).
    *   `index.html`: Homepage structure, includes container for heatmap (potentially populated by HTMX).
    *   `stock_detail.html`: Structure for stock detail page, includes containers for info, chart, timeframe controls.
    *   `index_detail.html` (Release 2): Structure for index detail page, includes containers for focused heatmap, top movers lists.
    *   `_tooltip.html`: HTML fragment template for the HTMX tooltip.
*   **`static/`:** Static assets.
    *   `css/custom.css`: Minimal custom CSS overrides or additions.
    *   `js/charting.js`: JavaScript for initializing and updating D3.js charts.
    *   `js/main.js` (Optional): Minimal site-wide JS if needed beyond charting/HTMX.

##### 5.2. Frontend (Browser)

*   **HTML Structure:** Rendered by Flask, using Bootstrap classes extensively for layout (containers, rows, columns), responsiveness, and basic component styling (buttons, cards).
*   **CSS Styling:** Primarily Bootstrap. `custom.css` used sparingly. Color coding (FR-11) applied dynamically via inline styles or CSS classes set by the backend (in templates or HTMX fragments) based on the defined scale (TBD).
*   **HTMX Interactions:**
    *   **Tooltip (FEAT-002):** Heatmap tiles have `hx-get="/api/tooltip/{symbol}"`, `hx-trigger="mouseenter[delay:200ms], focus"` (desktop) and potentially a tap-hold equivalent managed via minimal JS or custom HTMX trigger for mobile. `hx-swap="innerHTML"` targets a dedicated tooltip container.
    *   **Heatmap Loading (Optional):** The heatmap container could have `hx-get="/api/heatmap/Nifty 50" hx-trigger="load"` to load data dynamically after the main page structure renders.
*   **JavaScript (`charting.js`):**
    *   Initializes D3.js line chart(s) on the Stock Detail page (`/stock/{symbol}`).
    *   Fetches initial chart data (`/api/stock/{symbol}/chart?timeframe=1D`) on page load.
    *   Adds event listeners to timeframe selection buttons (FEAT-005).
    *   On timeframe button click:
        *   Prevents default button action.
        *   Fetches new chart data from `/api/stock/{symbol}/chart?timeframe=<selected_tf>` using `fetch()`.
        *   Updates the D3.js chart with the new data.
        *   Handles loading/error states during fetch.
    *   Ensures chart uses a neutral color (FR-4.3) and has appropriate axes (FR-4.5).
*   **Responsiveness (FEAT-010):** Achieved primarily through Bootstrap's grid system (`col-xs-*`, `col-md-*`, etc.) and responsive utility classes applied in the HTML templates. Charts need to be configured (D3.js) to resize appropriately within their containers.

#### 6. Data Flow Examples

*   **Homepage Load (MVP):**
    1.  User navigates to `/`.
    2.  Flask `routes.py` handles `/`.
    3.  Flask renders `index.html` template.
    4.  *Option A (Data Embedded):* Flask calls `data_service.py` -> `yfinance` -> gets Nifty 50 data -> calculates % changes -> passes data to `index.html` template -> Browser renders heatmap directly.
    5.  *Option B (HTMX Load):* Flask renders `index.html` with an empty heatmap container having `hx-get="/api/heatmap/Nifty 50" hx-trigger="load"`. Browser renders page -> HTMX triggers request -> Flask `api.py` handles `/api/heatmap/Nifty 50` -> `data_service.py` -> `yfinance` -> returns JSON -> HTMX receives JSON (or backend renders fragment) -> HTMX updates heatmap container. (Option B generally preferred for faster initial page load).
*   **Stock Tooltip Hover (MVP):**
    1.  User hovers over a tile for 'RELIANCE' on the heatmap.
    2.  HTMX detects `mouseenter` trigger on the tile element.
    3.  HTMX sends `GET` request to `/api/tooltip/RELIANCE`.
    4.  Flask `api.py` handles the request.
    5.  Calls `data_service.py` -> `yfinance` (likely cached) -> gets required data -> calculates % change, volume change.
    6.  Flask renders `_tooltip.html` fragment with the data.
    7.  Flask returns the HTML fragment.
    8.  HTMX receives the fragment and injects it into the designated tooltip display area.
    9.  User moves mouse away, tooltip disappears (handled by CSS or minimal JS if needed).
*   **Stock Detail Page Load & Chart Timeframe Change (MVP + R2):**
    1.  User clicks 'RELIANCE' tile on heatmap (standard link `href="/stock/RELIANCE"`).
    2.  Browser requests `/stock/RELIANCE`.
    3.  Flask `routes.py` handles the request.
    4.  Flask renders `stock_detail.html` template, passing `symbol='RELIANCE'`.
    5.  Browser renders the page structure.
    6.  `charting.js` executes on page load.
    7.  JS makes `fetch()` request to `/api/stock/RELIANCE/details` -> Flask `api.py` -> `data_service.py` -> returns JSON -> JS updates price info display.
    8.  JS makes `fetch()` request to `/api/stock/RELIANCE/chart?timeframe=1D` -> Flask `api.py` -> `data_service.py` -> `yfinance` -> returns JSON -> JS uses D3.js to render the initial 1D line chart.
    9.  User clicks the "6M" timeframe button (FEAT-005).
    10. `charting.js` event listener triggers.
    11. JS makes `fetch()` request to `/api/stock/RELIANCE/chart?timeframe=6M`.
    12. Flask `api.py` -> `data_service.py` -> `yfinance` (likely cached for recent data, fetches older if needed) -> returns JSON for 6 months.
    13. JS receives JSON and uses D3.js to update the existing chart with the new data.

#### 7. Addressing TBDs from PRD

This design requires clarification on the following points from the PRD. The architecture is flexible to accommodate the decisions:

*   **[Tile Sizing TBD] (FR-1.X, A5, D5):** Affects CSS/HTML structure within `index.html` and potentially `index_detail.html`. Backend API remains unchanged. Decision needed: Uniform or Weighted?
*   **[Max Definition TBD] (FR-5.4, D4):** Affects logic within `data_service.py` when handling the 'Max' timeframe request for `/api/stock/{symbol}/chart`. Decision needed: Since listing? Max available from `yfinance`? Specific date range?
*   **[Color Scale TBD] (FR-11.3, A3, D2):** Critical for UI. The specific % thresholds and corresponding hex/HSL codes need to be defined. This will be implemented via:
    *   Backend: A utility function used by API endpoints (`/api/heatmap`, `/api/tooltip`, `/api/index/.../top_movers`) to determine the appropriate color class/style based on % change.
    *   Frontend: Corresponding CSS rules in `custom.css` or inline styles applied in templates/fragments.
*   **[Data Timing TBD] (FR-1.4, FR-2.3, FR-3.2, NFR-07, A1, D1):** Affects when the backend data fetching cron job/task should run and what "Current Day Close" and "Previous Day Close" mean. Assumed to be End-of-Day data from `yfinance`. Needs confirmation. Backend `data_service.py` logic depends on this.
*   **[Tie-breaking Logic TBD] (FR-7.2, FR-9.1, FR-9.2, D8):** Affects logic within the `/api/index/{index_name}/top_movers` endpoint in `api.py`. Decision needed: Alphabetical? Higher Volume? etc.
*   **[Error Visuals TBD] (NFR-07, D6):** Affects frontend templates and potentially `charting.js`. How should loading states (spinners?) and API error messages be displayed? Requires standard UI patterns.
*   **[Disclaimer Placement TBD] (NFR-07, A2, D7):** Affects the `base.html` template. Decision needed: Footer? Separate About page?
*   **[Charting Library TBD] (D3):** PRD specifies D3.js. This design uses D3.js. If changed, `charting.js` and potentially dependencies would change.

#### 8. Release Plan Alignment

*   **MVP (Weeks 1-6):** Requires core Flask app structure, `/`, `/stock/{symbol}` routes, `/api/heatmap/Nifty 50`, `/api/tooltip/{symbol}`, `/api/stock/{symbol}/details`, `/api/stock/{symbol}/chart?timeframe=1D` endpoints. `data_service.py` for Nifty 50 and 1D data. Frontend templates (`index.html`, `stock_detail.html`, `_tooltip.html`), Bootstrap integration, HTMX for tooltips, `charting.js` for 1D D3 chart. Basic color coding logic (using placeholder scale if final TBD). Browser back navigation.
*   **Release 2 (Weeks 7-10):** Add `/index/{index_name}` route, `/api/index/{index_name}/top_movers` endpoint. Extend `/api/stock/{symbol}/chart` to handle all timeframes. Extend `data_service.py` for Nifty Next 50 and other timeframes. Add `index_detail.html` template. Update `charting.js` for timeframe selection. Implement explicit back navigation (e.g., simple link/button in templates). Integrate Nifty Next 50 into homepage heatmap API/template.
*   **Release 3 (Weeks 11-14):** Add `/api/index/{index_name}/summary` endpoint. Add corresponding UI elements to `index.html` (potentially loaded via HTMX). Implement refined error/loading states (TBD). Add final disclaimer/attribution (TBD).

#### 9. Future Considerations

*   **Scalability:** Deploy Flask using a production-grade WSGI server (Gunicorn) behind a reverse proxy (Nginx). Scale horizontally by adding more Flask instances. Use a robust caching backend (Redis). Serve static assets via CDN.
*   **Data Sources:** The `data_service.py` acts as an abstraction layer. If `yfinance` proves insufficient or alternative sources are needed, only this module needs significant changes, minimizing impact on API endpoints and frontend.
*   **Testing:** Implement unit tests for `data_service.py` logic and API endpoint responses. Frontend testing (e.g., using Playwright or Cypress) for UI interactions and responsiveness.
*   **Monitoring:** Integrate logging and monitoring (e.g., Sentry, Prometheus/Grafana) for error tracking and performance analysis.
