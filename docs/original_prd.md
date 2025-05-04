### Updated Product Requirements Document (with Placeholders for Later Detail)

Here is the PRD incorporating these recommendations and clearly marking areas needing further definition. Changes based on the investor perspective are noted.

```markdown
# Product Requirements Document: Visual Market Pulse Dashboard (Frontend) - v1.2

**Version:** 1.2
**Date:** 2025-05-05
**Status:** Updated Draft (Investor Perspective Review)

## 1. Introduction

This document outlines the frontend requirements for the Visual Market Pulse Dashboard, a web application designed to provide users (particularly those seeking a quick, long-term perspective) with an intuitive, visualization-driven overview of the Indian stock market, initially focusing on the Nifty 50 and Nifty Next 50 indices. The core philosophy emphasizes minimal text, direct interaction with visual elements, and rapid comprehension of market state and trends. Data is sourced via backend APIs, which in turn use Yahoo Finance (`yfinance`) exclusively for **daily (1D) or greater** historical technical data for this version. This PRD focuses exclusively on the client-side/frontend components and interactions.

## 2. Goals

*   Provide a highly scannable (< 30 seconds), visual overview of market health (Nifty 50, Nifty Next 50).
*   Enable intuitive drill-down into specific indices and individual stocks through direct interaction with visualizations.
*   Deliver a clean, minimalist, and responsive user experience across desktop and mobile devices using **Bootstrap**.
*   Ensure rapid comprehension of stock/index performance through consistent color-coding (red/green) based on defined intensity mappings.
*   Allow quick assessment of historical price trends via neutral-colored line charts.

## 3. Feature Definition & Prioritization (Kano Model)

| Feature ID | Feature Name    | Description    | Kano Category | Priority (MVP+) |
| :---- | :---- | :---- | :---- | :---- |
| FEAT-001   | Homepage Market Heatmap    | Visual grid of Nifty 50/Next 50 stocks, color-coded by daily % change. Tiles are interactive. **[Tile Sizing TBD]**    | Basic    | MVP    |
| FEAT-002   | Stock Tile Tooltip (HTMX)    | On hover/tap-hold on heatmap tile, display Stock Name, Daily % Change, Daily Volume Change via HTMX.    | Performance   | MVP    |
| FEAT-003   | Stock Detail Page View    | Dedicated page for a single stock showing basic info and price chart.    | Basic    | MVP    |
| FEAT-004   | Stock Price Chart    | **Neutral color** line chart on Stock Detail Page showing price trend (1D minimum granularity). Focus on trend shape.    | Basic    | MVP (Line)    |
| FEAT-005   | Chart Timeframe Selection    | Controls to select chart timeframe (e.g., 1D, 5D, 1M, 6M, 1Y, Max **[Max Definition TBD]**).    | Performance   | MVP (Controls), R2 (Functionality) |
| FEAT-007   | Index Detail Page View    | Dedicated page for an index (Nifty 50/Next 50) showing focused heatmap of **Top 5 Gainers/Losers**, plus lists.    | Performance   | Release 2    |
| FEAT-009   | Top Gainers/Losers Visuals    | Visual representation (e.g., lists) of **Top 5** movers on Index Detail Page.    | Performance   | Release 2    |
| FEAT-010   | Responsive Design (**Bootstrap**) | Fluid layout adaptation across desktop and mobile viewports using **Bootstrap**.    | Basic    | MVP    |
| FEAT-011   | Consistent Color Coding    | Strict use of red/green shades for negative/positive changes, intensity indicating magnitude per spec **[Scale TBD]**.    | Basic    | MVP    |
| FEAT-012   | Minimalist Navigation    | Back arrows and/or breadcrumbs for navigating between pages.    | Basic    | MVP (Browser Back), R2 (Explicit) |
| FEAT-014   | Overall Index Indicators (Home)  | Small visual gauges/bars on Homepage showing overall Nifty 50/Next 50 daily performance.    | Excitement    | Release 3    |

*(Note: Sector-level views, News Integration, Key Stock Metrics, Dark Mode, and Candlestick charts are considered future roadmap items beyond these initial releases).*

## 4. Functional & Non-Functional Requirements

### 4.1. Functional Requirements (Mapped to Features)

**FEAT-001: Homepage Market Heatmap**
*   **FR-1.1:** Display a grid representing stocks from Nifty 50.
*   **FR-1.2:** Display a grid representing stocks from Nifty Next 50 (Post-MVP).
*   **FR-1.3:** Each stock shall be represented by a rectangular tile.
*   **FR-1.X:** **[Heatmap Tile Sizing TBD - Uniform or Weighted?]** - *Decision needed on whether all tiles are equal size or weighted (e.g., by market cap).*
*   **FR-1.4:** Tile background color must be determined by the stock's **Daily Percentage Change** (fetched from API).
    *   **Definition:** Daily Percentage Change = `((Current Day Close Price - Previous Day Close Price) / Previous Day Close Price) * 100%`. **[Data Timing TBD]**
    *   Color mapping follows **FR-11.3**.
*   **FR-1.5:** Clicking/tapping a stock tile must navigate the user to the corresponding Stock Detail Page (**FEAT-003**).
*   **FR-1.6:** (Optional - **FEAT-014**) Clicking/tapping an overall index indicator must navigate to the Index Detail Page (**FEAT-007**).

**FEAT-002: Stock Tile Tooltip (HTMX)**
*   **FR-2.1:** On mouse hover over a stock tile (desktop), trigger an HTMX request/display.
*   **FR-2.2:** On tap-and-hold on a stock tile (mobile), trigger an HTMX request/display.
*   **FR-2.3:** The tooltip must display: Stock Name, **Daily Percentage Change**, **Daily Volume Change**.
    *   **Definition:** Daily Volume Change = `((Current Day Volume - Previous Day Volume) / Previous Day Volume) * 100%`. **[Data Timing TBD]**
*   **FR-2.4:** The tooltip must disappear when the hover/hold ends.

**FEAT-003: Stock Detail Page View**
*   **FR-3.1:** Display the Stock Name/Symbol prominently.
*   **FR-3.2:** Display basic current price information (e.g., Last Traded Price, Daily Change). **[Data Timing TBD]**
*   **FR-3.3:** Include the Stock Price Chart (**FEAT-004**).
*   **FR-3.4:** Include Chart Timeframe Selection controls (**FEAT-005**).
*   **FR-3.5:** Provide navigation back to the previous page (**FEAT-012**).

**FEAT-004: Stock Price Chart**
*   **FR-4.1:** Display a time-series chart representing the stock's price.
*   **FR-4.2:** Initial chart type: **Line chart (MVP)**.
*   **FR-4.3:** The line chart must use a **single, neutral color** (e.g., blue or theme color). *Rationale: Focuses user on trend shape, consistent with investor quick-scan goal. Red/green reserved for performance magnitude indicators.*
*   **FR-4.4:** Chart must display data corresponding to the selected timeframe (**FEAT-005**), starting with 1D (MVP).
*   **FR-4.5:** Chart axes must be clearly labeled (Time, Price).
*   **FR-4.6:** Chart must use **1 Day or greater granularity** for data points. No intraday data will be displayed.

**FEAT-005: Chart Timeframe Selection**
*   **FR-5.1:** Display interactive controls (e.g., buttons) labeled "1D", "5D", "1M", "6M", "1Y", "Max".
*   **FR-5.2:** Clicking/tapping a timeframe control must update the Stock Price Chart (**FEAT-004**) to display data for the selected period.
*   **FR-5.3:** The "1D" control must be active by default on page load (MVP). Other timeframes functional in Release 2.
*   **FR-5.4:** **[Definition of "Max" Timeframe TBD]** - *Needs clarification (e.g., since listing, max from yfinance, etc.).*

**FEAT-007: Index Detail Page View (Release 2)**
*   **FR-7.1:** Display the Index Name (e.g., "Nifty 50") prominently.
*   **FR-7.2:** Display a focused heatmap containing the **Top 5 Gaining and Top 5 Losing** stocks within that index based on Daily Percentage Change. Heatmap follows **FR-1.3** and color mapping **FR-11.3**. Tiles link to **FEAT-003**. **[Tie-breaking Logic TBD]** - *How are ties handled?*
*   **FR-7.3:** Include Top Gainers/Losers Visuals (**FEAT-009**).
*   **FR-7.4:** Provide navigation back to the Homepage via **FEAT-012**.

**FEAT-009: Top Gainers/Losers Visuals (Release 2)**
*   **FR-9.1:** Display the **top 5** gaining stocks within the index visually (e.g., simple list). **[Tie-breaking Logic TBD]**
*   **FR-9.2:** Display the **top 5** losing stocks within the index visually. **[Tie-breaking Logic TBD]**
*   **FR-9.3:** Each item should show Stock Symbol and Daily % Change, using red/green color coding per **FR-11.1/FR-11.2**.

**FEAT-010: Responsive Design (Bootstrap)**
*   **FR-10.1:** All pages must render correctly and be usable on common screen widths (e.g., 360px+, 768px+, 1024px+) using **Bootstrap's grid system and responsive utilities**.
*   **FR-10.2:** Layout elements (heatmaps, charts) must reflow or resize appropriately using Bootstrap classes.
*   **FR-10.3:** Interactions (click/tap targets, tooltips) must function correctly on touch and non-touch devices.
*   **FR-10.4:** Styling must be achieved primarily using **Bootstrap**. Custom CSS should be minimal and follow Bootstrap conventions where possible.

**FEAT-011: Consistent Color Coding**
*   **FR-11.1:** All representations of positive change (gain %) in heatmaps must use shades of green.
*   **FR-11.2:** All representations of negative change (loss %) in heatmaps must use shades of red.
*   **FR-11.3:** Color intensity **must** indicate the magnitude of change based on a defined scale. **[Exact Scale TBD - See D2]** - *This scale (specific % thresholds and corresponding hex/HSL codes) is critical and needs precise definition.* Example structure:
    *   `> +X%`: Brightest Green (`#TBD`)
    *   `+Y% to +X%`: Medium Green (`#TBD`)
    *   `> 0% to +Y%`: Light Green (`#TBD`)
    *   `0%`: Neutral Grey (`#TBD`)
    *   `< 0% to -Y%`: Light Red (`#TBD`)
    *   `-Y% to -X%`: Medium Red (`#TBD`)
    *   `< -X%`: Brightest Red (`#TBD`)
*   **FR-11.4:** Neutral states (0% change) should use a distinct neutral color (e.g., light grey). **[Exact Color TBD]**

**FEAT-012: Minimalist Navigation**
*   **FR-12.1:** MVP: Rely on browser's back button for navigation. *(Note: Consider adding explicit back button even in MVP for better UX).*
*   **FR-12.2:** Release 2+: Implement a clear "Back" arrow or breadcrumb trail (e.g., "Market Pulse > Nifty 50 > RELIANCE") on Detail pages.

**FEAT-014: Overall Index Indicators (Home) (Release 3)**
*   **FR-14.1:** Display small, separate visual indicators (e.g., colored bars or simple gauges) near the top of the homepage.
*   **FR-14.2:** One indicator for Nifty 50 overall daily % change, one for Nifty Next 50.
*   **FR-14.3:** Indicators must use the standard red/green color coding (**FR-11.1, FR-11.2**).

### 4.2. Non-Functional Requirements

*   **NFR-01: Performance** (As before)
*   **NFR-02: Usability** (As before, emphasizing scanability)
*   **NFR-03: Compatibility** (As before)
*   **NFR-04: Maintainability** (As before)
*   **NFR-05: Scalability (Frontend)** (As before)
*   **NFR-06: Security (Frontend)** (As before)
*   **NFR-07: Data Handling**
    *   Display data fetched from the backend API accurately.
    *   Provide clear visual indication if data is loading or unavailable. **[Error Visuals TBD]** - *How are errors/loading states shown?*
    *   Display data source attribution (**Yahoo Finance**) and any required disclaimers (e.g., delayed data, informational purposes only). **[Disclaimer Placement TBD]** - *Where exactly? Footer? About page?*
    *   **[Specific Data Update Timing TBD]** - *When is "daily" data refreshed (e.g., EOD time)?*

## 5. User Workflows & Journeys (User Story Mapping - Examples)

These workflows illustrate how a user, particularly one with a long-term investment horizon seeking a rapid market overview, might interact with the Visual Market Pulse Dashboard based on the defined features (MVP and subsequent releases).

Workflow 1: Quick Daily Market Health Check (MVP Focus)
Persona Goal: Get a < 30-second feel for the overall market (Nifty 50) direction and identify any significant outliers for potential deeper investigation later.
Steps:
User navigates to the dashboard homepage.
Instantly observes the Nifty 50 Market Heatmap (FEAT-001). The user primarily scans the overall color balance (predominance of red vs. green) and the intensity of colors (FR-11.3) to gauge market sentiment and volatility for the day.
The user spots a few tiles with particularly bright red or green colors, indicating strong movers.
They hover over (desktop) or tap-and-hold (mobile) one of these tiles (FEAT-002).
A tooltip appears via HTMX, showing the Stock Name, Daily % Change, and Daily Volume Change (FR-2.3), confirming the magnitude of the move without leaving the heatmap view.
The user repeats step 4-5 for another 1-2 outliers.
(Release 3 Addition): User glances at the Overall Index Indicators (FEAT-014) for a summarized Nifty 50/Next 50 daily change.
Outcome: The user has quickly assessed the day's market state for the Nifty 50 and noted key movers in under 30 seconds, fulfilling the core goal.
Workflow 2: Investigating a Stock's Daily Move in Long-Term Context (MVP + R2 Timeframes)
Persona Goal: Understand if a stock's significant daily movement (identified in Workflow 1) represents a blip or aligns with/deviates from its longer-term price trend.
Steps:
Following Workflow 1, the user identifies a stock (e.g., "Stock ABC") showing a significant daily change on the heatmap (FEAT-001).
The user clicks/taps the "Stock ABC" tile (FR-1.5).
They are navigated to the Stock Detail Page (FEAT-003).
The user notes the basic price info (Last Price, Daily Change) (FR-3.2).
They observe the Stock Price Chart (FEAT-004). Initially, it shows the "1D" view (FR-5.3), displayed as a neutral-colored line (FR-4.3). (Note: The utility of a 1D line chart is limited, but per PRD, it's the default. The value comes next).
The user interacts with the Chart Timeframe Selection controls (FEAT-005), clicking "6M", "1Y", or "Max" (Functionality R2, Controls MVP - FR-5.2, FR-5.4).
The neutral-colored line chart updates to show the selected longer-term historical price trend (FR-4.4).
The user assesses the shape and direction of the long-term trend, comparing it mentally to the day's significant move to gain context.
The user navigates back to the heatmap using the browser's back button (FEAT-012, FR-12.1) or an explicit back control (R2+).
Outcome: The user has contextualized a specific stock's daily performance against its historical trend, aiding in filtering out short-term noise from a long-term perspective.
Workflow 3: Reviewing Index Movers (Release 2 Focus)
Persona Goal: Understand which specific stocks are the primary drivers behind an index's (e.g., Nifty 50) overall movement for the day.
Steps:
User navigates to the dashboard homepage.
(Optional - Release 3): User clicks the overall Nifty 50 indicator (FEAT-014, FR-1.6) to go directly to the Index Detail Page.
(Alternative Navigation TBD): User navigates to the Nifty 50 Index Detail Page (FEAT-007).
The user views the focused heatmap showing only the Top 5 Gainers and Top 5 Losers for the index (FR-7.2). Color intensity highlights the magnitude.
The user also scans the explicit Top Gainers/Losers lists (FEAT-009) which show symbols and % change (FR-9.3).
The user identifies a stock of interest from these focused views (e.g., a Top Gainer).
They click the stock's tile on the focused heatmap (FR-7.2) or potentially an item in the list.
They are navigated to the Stock Detail Page (FEAT-003) for that specific stock.
The user follows steps 5-9 from Workflow 2 to analyze the stock's long-term trend.
The user navigates back (FEAT-012) to the Index Detail Page or Homepage.
Outcome: The user has identified the key stocks influencing the index's performance for the day and investigated one further within its long-term context.

## 6. Technical Feasibility & Architecture (Conceptual Frontend)

Stack Overview:
The frontend will be built using Flask (Python) as the backend framework, HTMX for dynamic HTML interactions, Bootstrap for responsive UI components, and D3.js for advanced data visualizations.

Architecture:

Flask serves as the lightweight backend, handling routing, API endpoints, and server-side rendering. It is well-suited for moderate traffic (20,000 hits/day) and can be easily scaled horizontally if needed.
HTMX enables partial page updates and dynamic content loading without full page reloads, reducing bandwidth and improving user experience. It integrates seamlessly with Flask’s templating.
Bootstrap provides a mobile-first, responsive design system, ensuring consistent UI/UX across devices with minimal custom CSS.
D3.js is used for interactive, investor-grade data visualizations (e.g., charts, graphs, dashboards), embedded within Bootstrap components for a cohesive look.

Scalability & Performance:

Flask’s stateless nature allows for easy scaling via WSGI servers (e.g., Gunicorn) and load balancers.
Caching (e.g., Flask-Caching, Redis) will be used for frequently accessed data and rendered templates.
Static assets (JS, CSS, images) will be served via CDN for low latency.
HTMX reduces server load by updating only necessary DOM fragments.
D3.js visualizations are rendered client-side, offloading processing from the server.

Security & Maintainability:
Follows Flask’s best practices for security (CSRF, XSS, input validation).
Modular codebase: separation of concerns between backend logic, frontend templates, and visualization scripts.
Bootstrap and HTMX minimize custom JS, reducing maintenance overhead.


## 7. Acceptance Criteria (Gherkin Syntax - Examples)

### FEAT-001: Homepage Market Heatmap
Feature: Homepage Market Heatmap

  Scenario: User views the homepage heatmap
    Given the user navigates to the homepage
    Then the user should see a grid of Nifty 50 stock tiles
    And each tile background color reflects the stock's daily percentage change using the defined red/green intensity scale

  Scenario: User clicks a stock tile
    Given the user is on the homepage heatmap
    When the user clicks a stock tile
    Then the user is navigated to the Stock Detail Page for that stock

### FEAT-002: Stock Tile Tooltip (HTMX)
Feature: Stock Tile Tooltip

  Scenario: User hovers over a stock tile (desktop)
    Given the user is on the homepage heatmap
    When the user hovers over a stock tile
    Then a tooltip appears showing Stock Name, Daily % Change, and Daily Volume Change

  Scenario: User tap-and-holds a stock tile (mobile)
    Given the user is on the homepage heatmap on a mobile device
    When the user tap-and-holds a stock tile
    Then a tooltip appears showing Stock Name, Daily % Change, and Daily Volume Change

### FEAT-003/004/005: Stock Detail Page & Chart
Feature: Stock Detail Page and Chart

  Scenario: User views a Stock Detail Page
    Given the user navigates to a Stock Detail Page
    Then the page displays the Stock Name, Symbol, and current price info
    And a neutral-colored line chart of the stock price for the default "1D" timeframe

  Scenario: User changes chart timeframe
    Given the user is on a Stock Detail Page
    When the user selects a different timeframe (e.g., "6M", "1Y")
    Then the line chart updates to show the price trend for the selected period

### FEAT-010: Responsive Design (Bootstrap)
Feature: Responsive Layout

  Scenario: User accesses the site on different devices
    Given the user accesses the dashboard on desktop, tablet, or mobile
    Then all pages and visualizations render correctly and remain usable

### FEAT-011: Consistent Color Coding
Feature: Color Coding

  Scenario: Heatmap color reflects stock performance
    Given the homepage heatmap is displayed
    Then positive changes use green shades, negative changes use red shades, and intensity matches the magnitude per the defined scale

### FEAT-012: Minimalist Navigation
Feature: Navigation

  Scenario: User navigates back from a detail page
    Given the user is on a Stock or Index Detail Page
    When the user uses the browser back button
    Then the user returns to the previous page

### FEAT-007/009: Index Detail Page & Top Movers (Release 2)
Feature: Index Detail Page and Top Movers

  Scenario: User views Index Detail Page
    Given the user navigates to an Index Detail Page
    Then the page displays a focused heatmap of the Top 5 Gainers and Top 5 Losers
    And lists of Top 5 Gainers and Losers with correct color coding and % change

### NFR-07: Data Handling & Error States
Feature: Data Loading and Error Handling

  Scenario: Data is loading
    Given the user navigates to any page
    When data is being fetched from the backend
    Then a loading indicator is shown

  Scenario: Data is unavailable or API error
    Given the user navigates to any page
    When the backend API returns an error or no data
    Then a clear error message or visual is displayed

### NFR: Data Source Attribution
Feature: Data Source Attribution

  Scenario: User views any page with market data
    Given the user is on any page displaying market data
    Then a visible attribution to Yahoo Finance is present

## 8. Release Strategy & Timeline (Incremental Roadmap)

### Focus: Deliver the core visual market scanning experience with minimal viable functionality

Timeline: Weeks 1-6
Key Deliverables:
Homepage Market Heatmap (FEAT-001) with Nifty 50 stocks
Basic Stock Tile Tooltip via HTMX (FEAT-002)
Simple Stock Detail Page (FEAT-003) with 1D price chart
Responsive Bootstrap implementation (FEAT-010)
Consistent color coding system (FEAT-011)
Browser back button navigation (FEAT-012)
Management Demo Value: Functional visual market scanner showing daily stock movements with intuitive color coding and basic interactivity. Demonstrates core product concept and visual approach.
Release 2: Enhanced Context & Analysis (4 weeks)

### Focus: Add timeframe flexibility and index-level insights

Timeline: Weeks 7-10
Key Deliverables:
Functional Chart Timeframe Selection (FEAT-005) - 5D, 1M, 6M, 1Y, Max
Index Detail Pages (FEAT-007) for Nifty 50
Top Gainers/Losers Visuals (FEAT-009)
Explicit back navigation controls (FEAT-012 enhancement)
Nifty Next 50 integration to Homepage Heatmap
Management Demo Value: Demonstrates product evolution with historical context capabilities and focused views of market movers. Shows how users can now analyze trends beyond daily movements.
Release 3: Market Context & Polish (4 weeks)

### Focus: Add overall market context and UX refinements

Timeline: Weeks 11-14
Key Deliverables:
Overall Index Indicators on Homepage (FEAT-014)
Enhanced error handling and loading states
Performance optimizations
Data source attribution and disclaimers
User feedback implementation from R1/R2
Management Demo Value: Complete market scanning solution with clear index-level indicators and polished user experience. Demonstrates product maturity and readiness for broader adoption.
Future Roadmap Considerations (Post-Release 3)
Sector-level views and filtering
News integration
Additional technical indicators
Dark mode support
Candlestick chart options
Mobile app consideration

## 9. Risk Management & Assumptions (RAID Log)

| Type    | ID | Description    | Impact (H/M/L) | Likelihood (H/M/L) | Mitigation / Contingency    | Owner    |
| :---- | :- | :---- | :---- | :---- | :---- | :---- |
| ... (Existing Risks R1-R5) ... |
| Assumption | A1 | Backend can provide performant APIs for all required data points sourced **exclusively from `yfinance` at 1D+ granularity**. **[Data Timing TBD]** | H    | -    | Ongoing communication with Backend team. Define clear API contracts & timing. | PM / FE Lead   |
| Assumption | A2 | The initial version will **exclusively use `yfinance`** via the backend API for all market data (1D+ granularity). Future iterations may explore alternative data sources. | M    | -    | Verify data characteristics. Include data delay disclaimers. **[Disclaimer Placement TBD]** | PM    |
| Assumption | A3 | The visual language, particularly the red/green color intensity mapping (**FR-11.3**), relies on common market conventions. **[Scale TBD]** Usability testing crucial. | M    | -    | Define mapping clearly (**D2**). Validate through usability testing. | PM / UX    |
| Assumption | A4 | HTMX provides sufficient interactivity for tooltips without conflicts or performance issues on target devices/browsers.    | L    | -    | Prototype HTMX interactions early. Test across devices.    | FE Lead    |
| Assumption | A5 | **[Heatmap Tile Sizing TBD]** - Assumption is uniform sizing unless specified otherwise. | L | - | Decide on sizing approach (uniform vs. weighted). | PM / UX |
| ... (Existing Issues I1) ... |
| Dependency | D1 | Backend API availability for Release 1 features (1D+ `yfinance` data). **[Data Timing TBD]** | H    | -    | Tracked via project plan; dependent on Backend team progress.    | PM    |
| Dependency | D2 | Finalized **specific color palette and intensity mapping scale** for red/green shades (**FR-11.3**). **[Scale TBD]** | H    | -    | UX/UI task, needed before extensive CSS implementation. Requires clear definition of ranges and corresponding hex codes/HSL values.    | UX / PM    |
| Dependency | D3 | Selection and approval of the JavaScript charting library. **[Library TBD]** | M    | -    | FE Lead to research and propose, PM/Team to approve.    | FE Lead / PM   |
| Dependency | D4 | Definition of "Max" timeframe for charts (**FR-5.4**). **[Definition TBD]** | L | - | Define based on data availability or product decision. | PM |
| Dependency | D5 | Decision on Heatmap Tile Sizing (**FR-1.X**). **[Sizing TBD]** | L | - | Decide uniform vs. weighted. | PM / UX |
| Dependency | D6 | Specification for Error/Loading Visuals (**NFR-07**). **[Error Visuals TBD]** | M | - | Define visual states. | UX / FE Lead |
| Dependency | D7 | Specification for Disclaimer Placement (**NFR-07**). **[Placement TBD]** | L | - | Decide placement (e.g., footer). | PM / UX |
| Dependency | D8 | Definition of Tie-Breaking Logic for Top G/L (**FR-7.2, FR-9.1, FR-9.2**). **[Logic TBD]** | L | - | Define how ties are handled. | PM / BE Lead |

```

