# Project Structure for Visual Market Pulse Dashboard

## Overview

The project is organized into a modular structure to promote maintainability and scalability. The root directory contains configuration files, documentation, and subdirectories for source code, scripts, and assets.

## Directory Structure

- **docs/**: Contains all documentation files, including this one.
  - architecture.md
  - tech-stack.md
  - project-structure.md (this file)
  - coding-standards.md
  - api-reference.md
  - data-models.md
  - environment-vars.md
  - testing-strategy.md
- **src/**: Source code for the application.
  - **backend/**: Flask application code.
    - app.py: Main entry point.
    - blueprints/: API endpoints organized by function.
    - services/: Business logic, e.g., yfinance_service.py.
  - **frontend/**: Static HTML, CSS, and JavaScript files.
    - index.html: Main page.
    - js/: Alpine.js components and scripts.
    - css/: Custom Bootstrap styles.
- **scripts/**: Utility scripts, e.g., for data processing or deployment.
- **tests/**: Unit and integration tests.
- **.taskmasterconfig**: Taskmaster configuration file.
- **.env**: Environment variables for sensitive data.

## File Organization Principles

- Keep files focused: One responsibility per file.
- Use descriptive names: E.g., `yfinance_service.py` for data fetching logic.
- Reference: See `docs/architecture.md` for how this structure integrates with the system.

This layout supports easy navigation and expansion as per `docs/coding-standards.md`.