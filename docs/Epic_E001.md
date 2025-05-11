**ID:** E001
**Date Created:** 2025-05-11
**Last Updated:** 2025-05-11
**Status:** Proposed
**Owner:** Product Owner (PO)
**Related PRD Section(s):** 3.1 (FEAT-001, FEAT-015, FEAT-002, FEAT-011), 4, 6.1

## 1. Epic Goal
*   Enable users like Priya to visually assess the End-of-Day (EOD) performance of the Nifty 50 and selected market sectors through interactive heatmaps. This epic focuses on providing clear visual cues for performance (color) and market capitalization (size), and allowing users to access basic stock details via tooltips, all using EOD data.

## 2. Business Value & Justification
*   This epic is the heart of our product. It directly tackles the core user problem of information overload and time-consuming EOD analysis (PRD 1.1) by offering a "visual-first approach" for a quick market pulse. By delivering this, we empower our users to make more informed decisions and save valuable time (PRD 1.1, 1.3). Successfully implementing this epic is fundamental to achieving key business goals, including user adoption of heatmap and sector features (PRD Goals 4 & 5) and improving average session duration (PRD Goal 2), as it delivers the primary value proposition of the dashboard. This is non-negotiable for the MVP.

## 3. Scope & Features Covered
*   **In Scope Features (from PRD):**
    *   FEAT-001: Homepage Market/Index Heatmap (EOD Data)
    *   FEAT-015: Sector Selection & Heatmap View (EOD Data)
    *   FEAT-002: Stock Tile Tooltip & Basic Info (EOD Data) (specifically tooltip content and its relation to heatmap data)
    *   FEAT-011: Consistent Color Coding & Legend (EOD Data)
*   **Key User Stories (from PRD Section 6.1):**
    *   US1.1: As Priya (long-term investor), I want to see the Nifty 50 stocks displayed as a heatmap by default when I open the dashboard, so I can get an immediate visual overview of the overall market's EOD performance.
    *   US1.2: As Priya, I want each stock tile in the Nifty 50 heatmap to be sized proportionally to its market capitalization relative to other Nifty 50 stocks, so I can quickly identify the EOD impact of larger companies.
    *   US1.3: As Priya, I want each stock tile in the Nifty 50 heatmap to be color-coded based on its EOD percentage change (shades of green for positive, red for negative, neutral for no change), so I can instantly gauge individual stock performance and overall market sentiment.
    *   US1.4: As Priya, I want to see the stock's ticker symbol on its heatmap tile (where space permits), so I can quickly identify familiar stocks.
    *   US1.5: As Priya, I want to see a clear list of available market sectors (e.g., Nifty Bank, Nifty IT) and an option for "Overall Market" (Nifty 50), so I can choose which group of stocks to analyze.
    *   US1.6: As Priya, when I select a specific market sector, I want the heatmap to update and display only the constituent stocks of that sector, so I can focus my EOD analysis on that part of the market.
    *   US1.7: As Priya, I want stock tiles within a sector heatmap to also be sized by their market cap (relative to other stocks *in that sector view*) and color-coded by their EOD percentage change, so I can apply the same visual analysis within a sector.
    *   US1.8: As Priya, I want the dashboard to clearly indicate which view is currently active (e.g., "Nifty 50" or "Nifty IT"), so I always have context for the displayed heatmap.
    *   US1.9: As Priya, when I hover over (desktop) or tap (mobile) any stock tile in any heatmap, I want a tooltip to appear showing the full stock name, ticker symbol, its precise EOD daily percentage change, and its EOD daily volume percentage change, so I can get quick, specific details without navigating away.
    *   US1.10: As Priya, I want to see a clear legend explaining the color-to-percentage change mapping (e.g., dark green = +5% or more), so I can accurately interpret the heatmap colors.
    *   US1.11: As a user, when heatmap data is being fetched or updated (e.g., on initial load or when switching sectors), I want to see a loading indicator, so I know the system is working and not frozen.
    *   US1.12: As a user, if heatmap data for the Nifty 50 or a selected sector fails to load, I want to see a user-friendly error message explaining the issue briefly, so I'm not left with a blank or broken screen.
*   **Out of Scope (for this Epic, clarify boundaries):**
    *   Real-time data updates (strictly EOD).
    *   User accounts, portfolio tracking, or watchlist functionality.
    *   Any historical price charts for individual stocks.
    *   Advanced charting features (e.g., technical indicators).
    *   Nifty Next 50 integration (planned for Release 2).
    *   Index Detail Pages showing Top Gainers/Losers (planned for Release 2).
    *   Heatmap tiles representing aggregated sector performance (MVP shows stocks *within* a sector, not sector-level aggregate tiles).
    *   The full application shell, detailed responsive design infrastructure beyond heatmap usability, and data handling aspects not directly tied to heatmap display (these are primarily covered in Epic E002).

## 4. Acceptance Criteria (High-Level for Epic)
*   Users can view a Nifty 50 heatmap by default upon landing, with stock tiles appropriately sized by market cap and color-coded by their EOD percentage change.
*   Users can easily select from a list of predefined market sectors and view a corresponding heatmap of that sector's constituent stocks, with tiles also sized and colored based on EOD data relative to that sector.
*   Interacting (hover/tap) with any stock tile in any active heatmap (Nifty 50 or sector) displays a tooltip containing the stock's full name, ticker symbol, precise EOD daily percentage change, and EOD daily volume percentage change.
*   A clear and understandable color legend explaining the EOD percentage change to color mapping (linear intensity scale) is always visible alongside the heatmaps.
*   The system provides clear loading indicators during data fetch for heatmaps and user-friendly error messages if data fails to load.
*   The visual cues (color, size, ticker symbol on tile) effectively convey EOD performance and significance, allowing for rapid comprehension with minimal cognitive load.

## 5. Dependencies
*   **Internal (Team/Technical):**
    *   Backend API endpoints (`/api/market/nifty50`, `/api/sectors`, `/api/sector/{sector_identifier}`) must be fully functional, providing data as per `api-reference.md` and `data-models.md`.
    *   The backend's server-side caching mechanism for EOD data must be operational to ensure performance and reliability (NFR-07).
    *   Frontend Alpine.js components for heatmap rendering, sector selection logic, and tooltip display logic must be developed.
*   **External (e.g., Data Sources, Other Teams):**
    *   D1 (PRD Section 10): Finalized list of market sectors to be supported in MVP. The backend team is to provide this based on yfinance data capabilities, with PM approval.
    *   D2 (PRD Section 10): Finalized specific color gradient (hex codes, percentage cutoffs for intensity steps) for heatmap color-coding (FR-11.3). This requires UX/UI input and PM approval.

## 6. Risks & Assumptions
*   **Key Risks (from PRD RAID Log, Section 10):**
    *   R1: yfinance API instability, rate limiting, or unexpected changes in its data structure could impact the availability or accuracy of data for heatmaps.
    *   R2: Potential performance issues when rendering large heatmaps or during frequent switching between sector views using Alpine.js, especially on less powerful devices.
    *   R3: Difficulty in accurately sourcing and mapping stocks to the correct Indian market sectors if `yfinance` does not provide this information directly or consistently.
    *   R4: Defining a linear color gradient scale (FR-11.3) that is both intuitive for users and meets accessibility standards across various percentage change values.
*   **Key Assumptions (from PRD RAID Log, Section 10):**
    *   A1: The backend can reliably fetch all necessary EOD data points (stock name, ticker, market cap, EOD % change, EOD volume % change) from `yfinance` for Nifty 50 and all specified sectors.
    *   A2: Market capitalization data, crucial for weighting tile sizes, is consistently available and accurate via `yfinance` EOD data.
    *   A3: The defined linear color gradient (once D2 is resolved) will be visually effective and accessible for users to interpret EOD performance.

## 7. Success Metrics (How will we measure success for this Epic?)
*   Directly contributes to PRD Business Goal 4: Achieve 50 users actively using the heatmap feature (overall market and/or sector-specific) at least once a week within the first 3 months post-launch.
*   Directly contributes to PRD Business Goal 5: Ensure at least 30% of active users engage with the sector selection feature within the first 3 months post-launch.
*   Qualitative feedback from MVP validation (PRD 3.3): Users confirm that the heatmaps (both market and sector views) are easy to understand, visually appealing, and useful for a quick EOD assessment.
*   Task completion rate during MVP validation (PRD 3.3): At least 80% of beta users successfully complete core user flows like viewing the Nifty 50 heatmap, selecting a sector, viewing the sector heatmap, and accessing stock info via tooltips without assistance.
*   User understanding during MVP validation (PRD 3.3): At least 75% of beta users demonstrate correct interpretation of the heatmap visualizations and color-coding for EOD data.

## 8. Technical Considerations (High-Level from PRD/Arch)
*   **Frontend:** Alpine.js will be used for dynamic heatmap rendering, managing sector selection state, and displaying tooltips. Bootstrap 5 will provide the foundational layout and styling for these components (PRD Section 7, `architecture.md`).
*   **Backend:** Flask will expose API endpoints to provide Nifty 50 data, lists of available sectors, and data for specific sectors (PRD Section 7, `architecture.md`).
*   **Data Source:** All data displayed will be End-of-Day (EOD), with 1-day granularity, sourced exclusively from `yfinance` via the backend (PRD Sections 1, 7).
*   **API Design:** Endpoints must be efficient, returning all necessary EOD data for a given view (Nifty 50 or a specific sector) in a single API response to minimize requests and latency (PRD Section 7, `api-reference.md`).
*   **Heatmap Logic:** The frontend will implement logic for a weighted heatmap layout where tile sizes are based on market capitalization relative to the current view. A linear color gradient scale for EOD percentage change needs to be consistently applied based on data from the backend (PRD Section 7, FR-1.4, FR-1.5, FR-11.3).
*   **State Management:** Client-side state (e.g., currently selected view, tooltip visibility, loading states, error states) will be managed using Alpine.js (`$store` for shared state, `x-data` for component-level state) (PRD Section 7).
*   **Performance (NFR-01):** Heatmap data for Nifty 50 and sectors should load within 3-4 seconds. Tooltip display should be near-instantaneous (<200ms), leveraging pre-fetched data.

## 9. Open Questions (Relevant to this Epic from PRD Section 12)
*   What are the exact hex codes and percentage mapping for the linear color gradient to be used for heatmaps, ensuring it's intuitive and meets accessibility guidelines? (Dependency D2)
*   How should market cap weighting be precisely calculated (e.g., linear scaling, logarithmic scaling) and applied for stock tiles within a sector view to ensure visual clarity and meaningful differentiation among stocks in that specific sector? (Architect/FE Lead to decide and prototype)
*   What is the specific strategy for handling missing EOD data for individual stocks within the Nifty 50 or a selected sector (e.g., display a greyed-out tile with "Data N/A", omit the tile entirely, or show an error icon on the tile)? This needs to be consistent. (FE Lead/BE Lead to define, PM to approve)
*   What is the specific minimum tile size threshold below which the stock's ticker symbol should be omitted from the tile itself (FR-1.7) to maintain readability? (FE Lead to determine based on visual testing)

## 10. Notes & Future Considerations
*   This epic delivers the core, tangible value to our users for the MVP. Its successful execution and positive reception are critical for the product's initial traction.
*   Future iterations could explore user-configurable heatmap parameters (e.g., sorting by percentage change, different metrics for tile sizing), additional visual cues, or even different types of visualizations.
*   Close attention must be paid to performance, especially if the number of stocks in sectors is large or if many sectors are added in the future.
