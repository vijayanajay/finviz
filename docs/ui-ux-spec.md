### UI/UX Specification: Visual Market Pulse Dashboard

### Introduction

This document defines the user experience goals, information architecture, user flows, and visual design specifications for the Visual Market Pulse Dashboard's user interface. The dashboard aims to provide users with an intuitive, visualization-driven overview of the Indian stock market (Nifty 50 and key sectors) using End-of-Day (EOD) data, with a strong emphasis on a heatmap-style visualization inspired by Finviz.

*   **Link to Primary Design Files:** (Placeholder: To be added once Figma/Sketch/XD files are created)
*   **Link to Deployed Storybook / Design System:** (Placeholder: To be added if a Storybook instance is created)

### Overall UX Goals & Principles

*   **Target User Personas:**
    *   Priya Sharma (30-45, Salaried Professional, Long-term investor using EOD data): Needs a quick (<1 minute) visual way to check EOD market (Nifty 50) and sector status, identify major EOD movers, and understand performance without complex charts or information overload. Values clean, uncluttered interfaces and time-saving visual summaries.
*   **Usability Goals:**
    *   **Rapid Comprehension:** Users should grasp the EOD market/sector state in under 30-60 seconds.
    *   **Ease of Learning:** The interface should be highly intuitive, requiring minimal to no instruction for interpreting EOD data visualizations.
    *   **Efficiency of Use:** Enable quick navigation between overall market and sector views. Tooltips should provide immediate access to essential stock details.
    *   **Error Prevention:** Clear visual cues and feedback should minimize misinterpretation of data.
    *   **Accessibility:** Adhere to WCAG 2.1 Level AA guidelines.
*   **Design Principles:**
    *   **Visual-First:** Prioritize heatmaps and color-coding over dense text for EOD data.
    *   **Speed to Insight:** Design for users to grasp EOD market and sector state rapidly.
    *   **Minimalism & Clarity:** Maintain a clean, uncluttered interface, avoiding overwhelming users.
    *   **Direct Interaction:** Allow users to interact directly with visual elements (e.g., tiles for tooltips).
    *   **Consistency:** Apply consistent color-coding, sizing logic, and interaction patterns across all views.

### Information Architecture (IA)

*   **Site Map / Screen Inventory:**
    The application is primarily a single-page application (SPA) with dynamic content updates. The main views are:
    1.  **Main Dashboard View:**
        *   Default: Nifty 50 Heatmap
        *   Alternative: Selected Sector Heatmap

    ```mermaid
    graph TD
        A[User Lands on Page] --> B(Main Dashboard View);
        B -- Default --> BH[Nifty 50 Heatmap Displayed];
        B -- User Action --> SS{Sector Selector};
        SS --> SH[Sector-Specific Heatmap Displayed];
        BH --> T[Stock Tile Interaction (Tooltip)];
        SH --> T;
    ```

*   **Navigation Structure:**
    *   **Primary Navigation:** Achieved through a "Sector Selector" element (e.g., dropdown or a list of buttons/tabs) prominently displayed, allowing users to switch between the "Overall Market" (Nifty 50) view and various "Sector-Specific" views.
    *   **Current View Indication:** The currently active view (e.g., "Nifty 50" or "Nifty IT") will be clearly indicated, likely as a heading above the heatmap or a highlighted state in the sector selector.
    *   **No deep hierarchical navigation** for MVP.

### User Flows

#### User Flow 1: Quick Daily Market & Sector Health Check

*   **Goal:** Allow Priya to quickly (< 1 minute) get a feel for the overall market (Nifty 50) and key sector EOD directions using visual cues.
*   **Steps / Diagram:**
    ```mermaid
    graph TD
        Start[User Navigates to Dashboard Homepage] --> NiftyView{Observe Default Nifty 50 Heatmap};
        NiftyView --> ScanNifty[Scans colors/sizes, reads tickers, notes legend & 'as of' date];
        ScanNifty --> SpotSelector[Spots Sector Selection UI];
        SpotSelector --> SelectSector[Selects "Nifty IT" from Sector List];
        SelectSector --> Loading1[Brief Loading Indicator];
        Loading1 --> ITView{Observe Nifty IT Heatmap};
        ITView --> ScanIT[Scans Nifty IT stocks, notes "Nifty IT" view indicator];
        ScanIT --> MaySelectAnother{Optionally Selects Another Sector?};
        MaySelectAnother -- Yes --> SelectAnotherSector[Selects "Nifty Bank"];
        SelectAnotherSector --> Loading2[Brief Loading Indicator];
        Loading2 --> BankView[Observe Nifty Bank Heatmap];
        BankView --> MaySwitchBack{Optionally Switch Back to Nifty 50?};
        MaySwitchBack -- Yes --> SelectNifty50[Selects "Overall Market"];
        SelectNifty50 --> Loading3[Brief Loading Indicator];
        Loading3 --> NiftyView;
        MaySelectAnother -- No --> InteractNiftyIT[Interact with Nifty IT Tile];
        InteractNiftyIT -- Hover/Tap --> ShowTooltipIT[Display Tooltip: Stock Name, Ticker, EOD % Change, EOD Volume % Change];
        ShowTooltipIT --> End;
        MaySwitchBack -- No --> InteractBank[Interact with Nifty Bank Tile];
        InteractBank -- Hover/Tap --> ShowTooltipBank[Display Tooltip: Stock Name, Ticker, EOD % Change, EOD Volume % Change];
        ShowTooltipBank --> End;
        ScanNifty --> InteractNifty50[Interact with Nifty 50 Tile];
        InteractNifty50 -- Hover/Tap --> ShowTooltipNifty50[Display Tooltip: Stock Name, Ticker, EOD % Change, EOD Volume % Change];
        ShowTooltipNifty50 --> End;
    ```

### Wireframes & Mockups

(Reference: `https://finviz.com/map.ashx` for heatmap style. Detailed wireframes/mockups to be created in a dedicated design tool.)

*   **Screen / View Name 1: Main Dashboard (Nifty 50 / Sector Heatmap)**
    *   **Layout:**
        *   **Header/Controls Area (Top):**
            *   Application Title/Logo (Subtle).
            *   Sector Selector (Dropdown or horizontal list of buttons/tabs). Clearly indicates "Overall Market (Nifty 50)" and available sectors (e.g., "Nifty Bank", "Nifty IT"). Active selection is highlighted.
            *   Color Legend: A small, always visible legend explaining the color intensity scale (e.g., gradient bar from dark red to dark green with % labels like "-5%", "0%", "+5%").
        *   **Main Content Area (Center):**
            *   Current View Title (e.g., "Nifty 50 EOD Pulse" or "Nifty IT EOD Pulse").
            *   Heatmap: A grid of rectangular tiles representing stocks.
                *   Tiles are sized proportionally by market capitalization (relative to other stocks in the *current* view). Larger cap = larger tile.
                *   Tiles are color-coded by EOD daily percentage change (Green for positive, Red for negative, Neutral for 0%). Intensity of color indicates magnitude.
                *   Each tile displays the stock's Ticker Symbol (e.g., "RELIANCE"). Font size adapts to tile size. For very small tiles, the ticker might be omitted (tooltip will have it).
        *   **Footer Area (Bottom):**
            *   "Data as of: [Date & Time]"
            *   Data Source Attribution: "Data sourced from Yahoo Finance via yfinance."
            *   Disclaimer: "Data is End-of-Day and for informational purposes only. Not financial advice."
    *   **Key Elements & Interactions:**
        *   **Sector Selector:** Clicking/tapping an option updates the Main Content Area to show the heatmap for the selected index/sector. A loading indicator appears during data fetch.
        *   **Heatmap Tiles:**
            *   Hover (Desktop) / Tap (Mobile): Displays a tooltip with Stock Name, Ticker Symbol, EOD Daily % Change, EOD Daily Volume % Change.
            *   Tooltip is non-intrusive and easily dismissible (e.g., mouse-out or tap outside).
        *   **Inspiration:** The Finviz treemap style where rectangles are packed together, sized by one metric (market cap) and colored by another (performance). The layout should be a simple grid or a treemap algorithm that fills the space efficiently.

### Component Library / Design System Reference

(To be detailed in Figma/Storybook. Below are key components based on PRD.)

#### Component Name: Heatmap Tile

*   **Appearance:** Rectangular. Background color determined by EOD % change (see Color Palette). Contains Ticker Symbol text.
    *   Inspired by Finviz tiles: clear, distinct blocks.
*   **States:**
    *   **Default:** Displays ticker, colored by performance.
    *   **Hover/Focus (Desktop):** May show a subtle border change or slight lift to indicate interactivity before tooltip appears. Tooltip appears.
    *   **Active/Tap (Mobile):** Tooltip appears.
*   **Behavior:** On hover/tap, triggers a Tooltip. Size is dynamic based on market cap relative to other tiles in the current view.

#### Component Name: Sector Selector

*   **Appearance:**
    *   Option A: Dropdown menu listing "Overall Market (Nifty 50)" and all available sectors.
    *   Option B: A horizontal row of buttons/tabs for "Overall Market (Nifty 50)" and key sectors.
    *   The selected/active view is clearly highlighted.
*   **States:**
    *   **Default:** Displays options.
    *   **Hover (for buttons/tabs):** Visual feedback.
    *   **Active/Selected:** Persistent visual distinction (e.g., different background, border, font weight).
*   **Behavior:** Selecting an option triggers an API call to fetch data for that view and updates the heatmap. A loading state should be indicated.

#### Component Name: Tooltip

*   **Appearance:** Small, non-intrusive overlay appearing near the interacted tile. Clean, legible text.
    *   Content: Stock Name, Ticker Symbol, Daily EOD % Change (e.g., "+1.25%"), Daily EOD Volume Change (e.g., "+15.5%").
*   **States:** Visible/Hidden.
*   **Behavior:** Appears on tile hover (desktop) or tap (mobile). Disappears on mouse-out (desktop) or tap outside (mobile). Data is pre-fetched with heatmap data for instant display.

#### Component Name: Color Legend

*   **Appearance:** A static visual guide. A horizontal gradient bar showing:
    *   Dark Red (e.g., <= -3%) -> Lighter Red -> Neutral Color (e.g., Grey for 0%) -> Lighter Green -> Dark Green (e.g., >= +3%).
    *   Labels for min, neutral, and max percentage points.
*   **States:** Static.
*   **Behavior:** Always visible when a heatmap is displayed, providing context for tile colors.

### Branding & Style Guide Reference

#### Color Palette

*   **Primary Performance Colors (Heatmap):**
    *   **Positive (Gain):** Shades of Green.
        *   Dark Green (Strong Gain, e.g., > +3%): `#107C10` (example)
        *   Medium Green (Moderate Gain): `#1DB954` (example)
        *   Light Green (Slight Gain): `#B2DFDB` (example)
    *   **Negative (Loss):** Shades of Red.
        *   Dark Red (Strong Loss, e.g., < -3%): `#D32F2F` (example)
        *   Medium Red (Moderate Loss): `#F44336` (example)
        *   Light Red (Slight Loss): `#FFCDD2` (example)
    *   **Neutral (0% Change):** Light Grey or Off-White.
        *   Example: `#E0E0E0` or `#F5F5F5`
*   **UI Element Colors:**
    *   **Background:** Light neutral (e.g., White `#FFFFFF` or very light grey `#FAFAFA`).
    *   **Text (Primary):** Dark Grey / Black (e.g., `#212121`).
    *   **Text (Secondary/Labels):** Medium Grey (e.g., `#757575`).
    *   **Borders/Dividers:** Light Grey (e.g., `#BDBDBD`).
    *   **Interactive Element Accent (e.g., selected sector):** A theme blue or a distinct neutral. Example: `#007BFF` (Bootstrap primary) or a darker grey.
*   **Color Mapping (FR-11.3):** A linear gradient scale. Specific hex codes and percentage cutoffs for intensity steps need to be finalized (Dependency D2 from PRD). The scale should be visually distinct and accessible. Finviz uses a fairly saturated palette; we can aim for something similar but ensure WCAG compliance.

#### Typography

*   **Font Families:** (To be decided, but prioritize readability and modern look. Standard sans-serif fonts are recommended, e.g., Inter, Roboto, Open Sans, or system fonts). Bootstrap defaults (e.g., system font stack) are a good starting point.
*   **Sizes & Weights:**
    *   **Ticker Symbols on Tiles:** Variable size based on tile dimensions. Bold weight for clarity.
    *   **Tooltip Text:** Small but legible (e.g., 12-14px). Regular weight.
    *   **Headings (e.g., "Nifty 50 EOD Pulse"):** Medium to Large (e.g., 18-24px). Semibold or Bold.
    *   **Sector Selector Text:** Regular (e.g., 14-16px).
    *   **Legend Text:** Small (e.g., 10-12px).
    *   **Footer Text:** Small (e.g., 10-12px).

#### Iconography

*   Minimal use of icons for MVP.
*   Potential icons:
    *   Dropdown arrow for sector selector (if dropdown style is chosen).
    *   Loading spinner icon.
*   Use a standard, clean icon set (e.g., Bootstrap Icons, Font Awesome if necessary, or simple SVGs).

#### Spacing & Grid

*   Leverage Bootstrap's grid system and spacing utilities (`m-`, `p-`) for consistent layout and spacing.
*   Ensure adequate padding within tiles, tooltips, and around interactive elements.
*   Maintain visual balance and avoid clutter.

### Accessibility (AX) Requirements

*   **Target Compliance:** WCAG 2.1 Level AA.
*   **Specific Requirements:**
    *   **Color Contrast:**
        *   Text on background (including text on heatmap tiles, tooltips, legend) must meet AA contrast ratios (4.5:1 for normal text, 3:1 for large text).
        *   Heatmap tile colors against their background (if any distinct background) and against the main page background.
        *   UI control colors (e.g., sector selector buttons) against their background.
    *   **Keyboard Navigation:**
        *   Sector selector must be fully navigable and operable using a keyboard (e.g., Tab to navigate, Enter/Space to select).
        *   If heatmap tiles become focusable for tooltip display, ensure a clear focus indicator and keyboard mechanism to trigger tooltips. (For MVP, hover/tap is primary; full keyboard on tiles might be post-MVP).
    *   **ARIA Landmarks/Attributes:**
        *   Use appropriate ARIA roles for main content areas, navigation (sector selector), and dynamic regions (heatmap updates).
        *   ARIA attributes for tooltips (`aria-describedby`) if applicable.
        *   ARIA attributes for loading states (`aria-busy`).
    *   **Text Alternatives:** Not directly applicable for heatmap visualization itself, but any icons used must have text alternatives.
    *   **Responsive Design:** Ensure content reflows and remains usable without horizontal scrolling on smaller screens.

### Responsiveness

*   **Breakpoints (Based on Bootstrap 5 defaults, can be customized):**
    *   **Mobile (Portrait):** <576px
    *   **Mobile (Landscape) / Small Tablet:** >=576px
    *   **Tablet:** >=768px
    *   **Desktop:** >=992px
    *   **Large Desktop:** >=1200px
*   **Adaptation Strategy:**
    *   **Heatmap:**
        *   Tiles will resize proportionally to fit the available width while maintaining aspect ratios or adapting them slightly.
        *   The number of tiles visible without excessive scrolling should be optimized. On very small screens, the heatmap might become vertically scrollable if all tiles cannot fit with minimum legibility.
        *   Ticker symbol font size on tiles will adapt. May be hidden on very small tiles to prevent clutter, relying on the tooltip.
    *   **Sector Selector:**
        *   If button-based, may wrap to two lines or convert to a dropdown on smaller screens.
        *   Dropdowns are inherently mobile-friendly.
    *   **Tooltips:** Ensure tooltips are positioned correctly and do not go off-screen on any device size. They should be easily dismissible on touch devices.
    *   **Legend & Footer:** Font sizes may adjust slightly. Layout should remain clean.
    *   Tap targets for interactive elements (tiles, sector selectors) must be adequately sized for touchscreens (e.g., min 44x44 CSS pixels).

### Change Log

| Change        | Date       | Version | Description                                      | Author         |
|---------------|------------|---------|--------------------------------------------------|----------------|
| Initial draft | 2025-05-11 | 0.1     | Initial draft based on PRD and template.         | Gemini 2.5 Pro |