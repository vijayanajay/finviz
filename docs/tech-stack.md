# Technology Stack for Visual Market Pulse Dashboard

## Overview

The Visual Market Pulse Dashboard utilizes a modern, lightweight technology stack to deliver an efficient and responsive user experience. This stack is chosen for its simplicity, performance, and ease of deployment, aligning with the project's MVP requirements as outlined in the PRD.

## Frontend Technologies

- **Alpine.js**: A minimal JavaScript framework for building interactive UI components without a full SPA setup. It handles dynamic elements like heatmaps and tooltips efficiently.
  - **Why?**: Lightweight (under 10kB), no build step required, perfect for rapid development and integration with HTML.
- **Bootstrap 5**: CSS framework for responsive design, grid layouts, and pre-built components (e.g., cards, modals).
  - **Why?**: Provides a mobile-first approach and ensures cross-browser consistency with minimal custom CSS.

## Backend Technologies

- **Flask (Python)**: Micro web framework for building the API server. It handles routing, request processing, and data serving.
  - **Why?**: Simple and flexible for MVP, with excellent support for extensions like Flask-Caching.
- **yfinance Library**: Python library for fetching financial data from Yahoo Finance.
  - **Why?**: Free, easy to use, and sufficient for EOD data needs without requiring a paid API.
- **Flask-Caching**: In-memory or Redis-based caching mechanism to store EOD data and reduce API calls.
  - **Why?**: Improves performance by caching frequently accessed data, with fallback to file-based caching for simple deployments.

## Data Storage and Caching

- **Caching Mechanism**: Primarily Redis (if available on Render), with file-based caching as a fallback.
  - **Why?**: Ensures data persistence and quick access for EOD updates.
- **No Persistent Database**: For MVP, data is fetched on-demand and cached; no SQL/NoSQL database is used to keep things simple.

## Deployment and Infrastructure

- **Docker**: Containerization for the Flask app, ensuring consistent environments across development and production.
  - **Why?**: Facilitates easy deployment on Render and handles dependencies seamlessly.
- **Render**: Cloud platform for hosting the backend (as a web service) and frontend (as static files).
  - **Why?**: Cost-effective, auto-scales, and integrates well with Docker.

## Development Tools

- **VS Code or Cursor**: IDE for development, with extensions for Python, JavaScript, and Markdown.
- **Git**: Version control for tracking changes and collaborating.
- **npm/yarn**: For managing any frontend dependencies, though minimized in this stack.

This stack supports the project's NFRs, such as fast load times and easy maintenance. For detailed usage, refer to `docs/coding-standards.md` and `docs/architecture.md`.