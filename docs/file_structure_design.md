## Detailed File and Method Breakdown

### Backend (Flask Application)

1.  **`run.py`** (or similar entry point)
    *   Purpose: Main entry point to start the Flask development server or production WSGI server.
    *   `main()`:
        *   Imports the application factory (`create_app`).
        *   Creates the Flask app instance using `create_app()`.
        *   Runs the app (`app.run()` for development or hands off to WSGI server like Gunicorn).

2.  **`app/`** (Application Package)
    *   **`__init__.py`**
        *   Purpose: Application factory. Initializes Flask app and extensions.
        *   `create_app(config_class=Config)`:
            *   Initializes `Flask(__name__)`.
            *   Loads configuration from `config.py` or environment variables.
            *   Initializes extensions (e.g., `cache.init_app(app)`).
            *   Registers Blueprints (e.g., `main_routes`, `api_routes`).
            *   Returns the configured Flask app instance.

    *   **`config.py`**
        *   Purpose: Store application configuration.
        *   `class Config:`:
            *   `SECRET_KEY`: For session management, CSRF protection (best practice).
            *   `CACHE_TYPE`: E.g., 'SimpleCache', 'RedisCache'.
            *   `CACHE_DEFAULT_TIMEOUT`: Default cache duration.
            *   `NIFTY_50_SYMBOLS`: List of stock symbols for Nifty 50.
            *   `NIFTY_NEXT_50_SYMBOLS`: List of stock symbols for Nifty Next 50 (Release 2+).
            *   `YAHOO_FINANCE_API_PARAMS`: Any potential parameters needed for `yfinance` calls (e.g., user agent if needed).
            *   `COLOR_SCALE_CONFIG`: **[Scale TBD]** Placeholder for the defined mapping of % change ranges to CSS classes or color codes (e.g., a dictionary). Used by `data_service.get_color_class`.
            *   `MAX_TIMEFRAME_DEFINITION`: **[Definition TBD]** Parameter defining 'Max' (e.g., 'max', specific start date). Used by `data_service.get_stock_history`.

    *   **`routes.py`** (or `main_routes.py` within a 'main' blueprint)
        *   Purpose: Define routes for serving main HTML pages.
        *   `@main_bp.route('/')`: `index()`
            *   Renders the `index.html` template.
            *   *Decision Point:* As per Design Option B (HTMX Load), this route primarily renders the *structure* of the page, relying on client-side JavaScript (specifically `heatmap.js`) to fetch data from `/api/heatmap/...` and populate the content dynamically.
        *   `@main_bp.route('/stock/<symbol>')`: `stock_detail(symbol)`
            *   Renders the `stock_detail.html` template, passing the `symbol` variable to the template.
        *   `@main_bp.route('/index/<index_name>')` (Release 2): `index_detail(index_name)`
            *   Renders the `index_detail.html` template, passing the `index_name` (e.g., 'Nifty 50').

    *   **`api.py`** (or `api_routes.py` within an 'api' blueprint)
        *   Purpose: Define API endpoints for data retrieval (JSON) or HTML fragments (HTMX). Handle potential errors and return appropriate responses (e.g., 404, 500).
        *   `@api_bp.route('/heatmap/<index_name>')`: `get_heatmap_data(index_name)`
            *   Calls `data_service.get_index_constituent_data(index_name)` to get symbols.
            *   Calls `data_service.get_batch_daily_change(symbols)` to get % change for all symbols.
            *   For each symbol, determines the appropriate CSS color class using `data_service.get_color_class()`.
            *   Formats data as JSON: `[{'symbol': 'XYZ', 'change': 1.5, 'color_class': 'change-pos-high'}, ...]`. (Covers FR-1.1, FR-1.2, FR-1.4).
            *   **Returns JSON response.**
            *   Handles potential errors from `data_service` (e.g., index not found, data fetch errors) and returns appropriate HTTP status codes/messages.
        *   `@api_bp.route('/tooltip/<symbol>')`: `get_tooltip_data(symbol)`
            *   Calls `data_service.get_stock_tooltip_info(symbol)` to get name, daily % change, daily volume %.
            *   Renders the `_tooltip.html` fragment template, passing the fetched data. (Covers FEAT-002, FR-2.3).
            *   **Returns HTML fragment response.** (This endpoint remains HTMX-driven).
            *   Handles potential errors (e.g., symbol not found).
        *   `@api_bp.route('/stock/<symbol>/details')`: `get_stock_details(symbol)`
            *   Calls `data_service.get_stock_current_info(symbol)` for name, LTP, daily change.
            *   Formats data as JSON: `{'name': '...', 'symbol': '...', 'price': ..., 'change': ...}`. (Covers FR-3.1, FR-3.2).
            *   **Returns JSON response.**
            *   Handles potential errors (e.g., symbol not found).
        *   `@api_bp.route('/stock/<symbol>/chart')`: `get_stock_chart_data(symbol)`
            *   Gets `timeframe` parameter from request query args (`request.args.get('timeframe', default='1D')`).
            *   Validates timeframe against allowed values ('1D', '5D', '1M', '6M', '1Y', 'Max'). Returns error if invalid.
            *   Calls `data_service.get_stock_history(symbol, timeframe)` using **1D or greater granularity**.
            *   Formats data as JSON: `{'timestamps': [...], 'prices': [...]}`. (Covers FEAT-004, FR-4.4, FR-5.2).
            *   **Returns JSON response.**
            *   Handles potential errors (e.g., symbol not found, data fetch errors).
        *   `@api_bp.route('/index/<index_name>/top_movers')` (Release 2): `get_index_top_movers(index_name)`
            *   Calls `data_service.get_index_constituent_data(index_name)`.
            *   Calls `data_service.get_batch_daily_change(symbols)`.
            *   Determines Top 5 Gainers/Losers based on % change. Includes **[Tie-breaking Logic TBD]** (e.g., handle ties by volume or alphabetically). Includes color class for each mover.
            *   Formats data as JSON: `{'gainers': [{'symbol': 'A', 'change': 2.1, 'color_class': '...'}, ...], 'losers': [...]}`. (Covers FR-7.2, FEAT-009, FR-9.1, FR-9.2, FR-9.3).
            *   **Returns JSON response.**
            *   Handles potential errors.
        *   `@api_bp.route('/index/<index_name>/summary')` (Release 3): `get_index_summary(index_name)`
            *   Gets the appropriate index ticker symbol (e.g., '^NSEI' for Nifty 50).
            *   Calls `data_service.get_index_daily_change(index_name)`. Includes color class for the index change.
            *   Formats data as JSON: `{'index': index_name, 'change': 0.5, 'color_class': '...'}`. (Covers FEAT-014, FR-14.2).
            *   **Returns JSON response.**
            *   Handles potential errors.

    *   **`data_service.py`**
        *   Purpose: Encapsulate all interactions with `yfinance` and data calculations. Apply caching.
        *   `_fetch_yf_history(ticker_symbol, period, interval='1d')`: (Internal helper)
            *   Uses `yfinance.Ticker(ticker_symbol).history(period=period, interval=interval)`.
            *   Handles potential `yfinance` exceptions (network errors, invalid ticker).
            *   Returns pandas DataFrame or None/empty on error.
        *   `_calculate_daily_change(df)`: (Internal helper)
            *   Takes DataFrame with 'Close' prices.
            *   Calculates daily % change based on **[Data Timing TBD]** definition (e.g., `df['Close'].pct_change().iloc[-1] * 100`). Requires clarity on what constitutes "Current Day" and "Previous Day" based on data availability.
            *   Returns float or None.
        *   `_calculate_volume_change(df)`: (Internal helper)
            *   Takes DataFrame with 'Volume'.
            *   Calculates daily volume % change based on **[Data Timing TBD]** definition. Requires clarity on what constitutes "Current Day" and "Previous Day" volume.
            *   Returns float or None.
        *   `@cache.memoize()`: `get_stock_history(symbol, timeframe)`:
            *   Maps `timeframe` ('1D', '5D', '1M', '6M', '1Y', 'Max') to appropriate `period` for `yfinance` (e.g., '1d', '5d', '1mo', '6mo', '1y', uses `MAX_TIMEFRAME_DEFINITION` from config for 'Max').
            *   Calls `_fetch_yf_history(symbol, period=mapped_period, interval='1d')`. Ensures **1D+ granularity**.
            *   Processes DataFrame to extract timestamps and closing prices.
            *   Returns dict `{'timestamps': [...], 'prices': [...]}` or empty/error structure.
        *   `@cache.memoize()`: `get_stock_current_info(symbol)`:
            *   Calls `yfinance.Ticker(symbol).info` (potentially selectively fetching needed fields).
            *   *Alternatively/Robustly:* Fetches 2 days history (`_fetch_yf_history(symbol, period='2d')`) to get last close and calculate daily change (`_calculate_daily_change`).
            *   Extracts Name, Last Traded Price (LTP), calculates Daily Change %.
            *   Returns dict `{'name': ..., 'symbol': ..., 'price': ..., 'change': ...}`.
        *   `@cache.memoize()`: `get_stock_tooltip_info(symbol)`:
            *   Requires name, daily % change, daily volume %.
            *   Calls `yfinance.Ticker(symbol).info` for name.
            *   Calls `_fetch_yf_history(symbol, period='2d')` (or longer if needed for volume average).
            *   Calculates daily % change (`_calculate_daily_change`).
            *   Calculates daily volume % change (`_calculate_volume_change`).
            *   Returns dict `{'name': ..., 'change': ..., 'volume_change': ...}`.
        *   `@cache.memoize()`: `get_index_constituent_data(index_name)`:
            *   Retrieves the list of symbols for the index from `config.py`.
            *   Returns list `['SYMBOL1', 'SYMBOL2', ...]`.
            *   *(Note: This assumes static lists from config. Dynamic fetching is out of scope per PRD/Design).*
        *   `get_batch_daily_change(symbols)`:
            *   Iterates through symbols list.
            *   For each symbol, attempts to get cached 2-day history (or fetches if needed: `_fetch_yf_history(symbol, period='2d')`).
            *   Calculates daily % change (`_calculate_daily_change`).
            *   Leverages caching for individual symbols.
            *   Returns dict `{'SYMBOL1': 1.5, 'SYMBOL2': -0.5, ...}`.
        *   `@cache.memoize()`: `get_index_daily_change(index_name)`:
            *   Gets the appropriate index ticker symbol (e.g., '^NSEI' for Nifty 50).
            *   Calls `_fetch_yf_history(index_ticker, period='2d')`.
            *   Calculates daily % change (`_calculate_daily_change`).
            *   Returns float (the % change).
        *   `get_color_class(change_percentage)`: (Utility function)
            *   Takes a float (% change).
            *   Uses `COLOR_SCALE_CONFIG` **[Scale TBD]** from `config.py` to map the percentage to a predefined range.
            *   Returns the appropriate CSS class name (e.g., 'change-pos-high', 'change-neg-low', 'change-neutral').

    *   **`templates/`**
        *   **`base.html`**:
            *   Basic HTML structure (`<html>`, `<head>`, `<body>`).
            *   `<head>` includes:
                *   `<meta>` tags (charset, viewport).
                *   `<title>`.
                *   Bootstrap CSS link (CDN or local).
                *   Link to `custom.css`.
            *   `<body>` includes:
                *   Minimal Header/Navbar structure (if any).
                *   Main content block (e.g., `{% block content %}{% endblock %}`).
                *   Footer area:
                    *   Copyright info.
                    *   Data source attribution (**Yahoo Finance**).
                    *   Disclaimer text (**[Placement TBD]** - Footer is a likely place).
                *   Bootstrap JS bundle link (CDN or local).
                *   HTMX script link (CDN or local).
                *   Link to `d3.min.js` (D3.js library).
                *   Link to `charting.js`.
                *   Link to `heatmap.js`.
                *   Link to `main.js` (if used).
        *   **`index.html`**:
            *   Extends `base.html`.
            *   `{% block content %}`:
                *   Container for Overall Index Indicators (Release 3) (**FEAT-014**). May use client-side JS to load from `/api/index/.../summary`.
                *   Container for Nifty 50 Heatmap (`id="nifty50-heatmap"`). This container will be populated by `heatmap.js` fetching data from `/api/heatmap/Nifty 50`. Show loading indicator **[Error Visuals TBD]** while loading.
                    *   *Tiles (rendered by `heatmap.js`):* Structure for tiles (e.g., `div` elements). Color class applied by JavaScript based on the `color_class` received in the JSON from the backend. `<a>` tag wrapping tile linking to `/stock/<symbol>`. HTMX attributes for tooltip (`hx-get="/api/tooltip/<symbol>"`, `hx-trigger="mouseenter[delay:200ms], focus"` etc., `hx-target="#stock-tooltip"`, `hx-swap="innerHTML"`). **[Tile Sizing TBD]** affects CSS classes/styles applied by JS.
                *   Container for Nifty Next 50 Heatmap (`id="niftynext50-heatmap"`) (Release 2+). Similar structure, populated by `heatmap.js` fetching data from `/api/heatmap/Nifty Next 50`.
                *   Container for the tooltip display (`id="stock-tooltip"`), positioned appropriately via CSS. This container is the `hx-target` for tooltip HTMX requests.
        *   **`stock_detail.html`**:
            *   Extends `base.html`.
            *   `{% block content %}`:
                *   Navigation: Explicit "Back" button/link (Release 2+) or rely on browser back (MVP). (**FEAT-012**).
                *   Container for Stock Info (`id="stock-info"`): Displays Name, Symbol, Price, Change (populated by `charting.js` via `/api/stock/<symbol>/details`).
                *   Container for Chart (`id="stock-chart-container"`): Fixed height/width container for the D3 chart.
                *   Container for Timeframe Selection Controls (`id="chart-timeframe-controls"`): Buttons ('1D', '5D', '1M', '6M', '1Y', 'Max') (**FEAT-005**).
        *   **`index_detail.html`** (Release 2):
            *   Extends `base.html`.
            *   `{% block content %}`:
                *   Navigation: Explicit "Back" button/link.
                *   Index Name heading.
                *   Container for Focused Heatmap (`id="index-focused-heatmap"`): Shows Top 5 Gainers/Losers tiles. Loaded via JS from `/api/index/<index_name>/top_movers`. Tiles link to stock detail. Color classes applied by JS based on backend JSON.
                *   Container for Top Gainers List (`id="top-gainers-list"`): Populated via JS.
                *   Container for Top Losers List (`id="top-losers-list"`): Populated via JS.
        *   **`_tooltip.html`**:
            *   HTML fragment (no `<html>`, `<body>` tags).
            *   Contains structure to display: Stock Name, Daily % Change, Daily Volume Change passed from the `/api/tooltip/<symbol>` endpoint. Styled using Bootstrap utility classes.

    *   **`static/`**
        *   **`css/custom.css`**:
            *   Minimal custom styles, Bootstrap overrides if necessary.
            *   CSS rules for the color intensity scale (**[Scale TBD]**): These classes correspond to the output of `data_service.get_color_class` and are applied by client-side JS.
                *   `.change-pos-high { background-color: #TBD_BRIGHT_GREEN; color: #TBD_TEXT; }`
                *   `.change-pos-med { background-color: #TBD_MEDIUM_GREEN; color: #TBD_TEXT; }`
                *   ...and so on for light green, neutral grey, light/medium/bright red.
            *   Styles for heatmap tile layout/sizing (**[Tile Sizing TBD]**).
            *   Styles for tooltip appearance and positioning.
            *   Styles for loading indicators / error messages (**[Error Visuals TBD]**).
            *   Styles for chart container sizing.
        *   **`js/d3.min.js`**: The D3.js library.
        *   **`js/charting.js`**:
            *   Purpose: Handle D3.js chart rendering and updates on the stock detail page.
            *   `initStockPage(symbol)`:
                *   Called on stock detail page load.
                *   Calls `fetchStockDetails(symbol)`.
                *   Calls `initStockChart(symbol)` with default timeframe ('1D').
                *   Sets up event listeners for timeframe buttons, calling `handleTimeframeClick`.
            *   `fetchStockDetails(symbol)`:
                *   Uses `fetch()` to GET `/api/stock/<symbol>/details`.
                *   Updates the DOM elements in `#stock-info` container with response data.
                *   Handles loading/error states visually **[Error Visuals TBD]**.
            *   `fetchChartData(symbol, timeframe)`:
                *   Uses `fetch()` to GET `/api/stock/<symbol>/chart?timeframe=<timeframe>`.
                *   Handles loading/error states visually **[Error Visuals TBD]**.
                *   Returns promise resolving with JSON data `{'timestamps': [...], 'prices': [...]}`.
            *   `initStockChart(symbol, initialTimeframe='1D')`:
                *   Selects the chart container (`#stock-chart-container`).
                *   Sets up D3.js SVG canvas, axes (Time, Price), scales.
                *   Configures chart appearance (e.g., **neutral line color** FR-4.3).
                *   Calls `fetchChartData(symbol, initialTimeframe)`.
                *   On data received, calls `renderChart` to draw the initial line.
                *   Ensures chart is responsive to container size changes.
            *   `renderChart(data)`: (or `updateChart`)
                *   Takes `{'timestamps': [...], 'prices': [...]}` data.
                *   Updates D3 scales based on new data domains.
                *   Updates axes.
                *   Selects the line path, binds new data, updates the 'd' attribute with transition. Renders the neutral colored line.
            *   `handleTimeframeClick(event)`:
                *   Gets the `symbol` (e.g., from a data attribute) and selected `timeframe` from the clicked button.
                *   Prevents default button action.
                *   Visually indicates loading state **[Error Visuals TBD]**.
                *   Calls `fetchChartData(symbol, timeframe)`.
                *   On data received, calls `renderChart` (or `updateChart`) to update the visualization.
        *   **`js/heatmap.js`**:
            *   Purpose: Fetch heatmap data (JSON) and render the heatmap tiles dynamically using vanilla JS.
            *   `initHeatmaps()`:
                *   Called on homepage load.
                *   Identifies heatmap containers (e.g., `#nifty50-heatmap`, `#niftynext50-heatmap`).
                *   For each container, determines the index name (e.g., 'Nifty 50').
                *   Calls `fetchHeatmapData(indexName)` for each.
            *   `fetchHeatmapData(indexName)`:
                *   Uses `fetch()` to GET `/api/heatmap/<indexName>`.
                *   Handles loading/error states visually **[Error Visuals TBD]** within the container.
                *   Returns promise resolving with JSON data `[{'symbol': 'XYZ', 'change': 1.5, 'color_class': '...'}, ...]`.
            *   `renderHeatmap(containerId, data)`:
                *   Selects the target container (`#containerId`).
                *   Clears existing content.
                *   Iterates through the `data` array.
                *   For each stock object:
                    *   Creates HTML elements (e.g., `div` for tile, `a` for link, `span` for symbol).
                    *   Applies the `color_class` from the data to the tile element.
                    *   Sets the link `href` to `/stock/<symbol>`.
                    *   Adds necessary HTMX attributes for the tooltip (`hx-get`, `hx-trigger`, `hx-target`, `hx-swap`).
                    *   Applies CSS classes for tile sizing/layout (**[Tile Sizing TBD]**).
                    *   Appends the created tile element to the container.
        *   **`js/main.js`** (Optional)
            *   Purpose: Minimal global JS, if needed.
            *   *Potential Use:* Initialize `initHeatmaps()` and `initStockPage()` based on the current page. Add custom event listener for 'taphold' on mobile for tooltips if standard HTMX triggers (`mouseenter`, `focus`) are insufficient. Could initialize Bootstrap components if needed.

