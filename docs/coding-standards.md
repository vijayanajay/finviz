# Coding Standards for Visual Market Pulse Dashboard

## Overview

These standards ensure consistent, maintainable, and readable code across the project. They align with best practices for Python (Flask) and JavaScript (Alpine.js), promoting collaboration and reducing errors.

## General Guidelines

- **Consistency**: Follow PEP 8 for Python and the Airbnb JavaScript style guide where applicable.
- **Readability**: Use meaningful variable names, add comments for complex logic, and keep functions under 50 lines.
- **Version Control**: Commit atomic changes with descriptive messages (e.g., "feat/backend: Add caching for EOD data").

## Python (Flask Backend)

- **Imports**: Group imports as standard, third-party, then local. Example:
  ```python
  import os
  from flask import Flask
  from services.yfinance_service import fetch_eod_data
  ```
- **Functions**: Use type hints and docstrings. Example:
  ```python
  def fetch_eod_data(symbol: str) -> dict:
      """Fetch EOD data for a given stock symbol using yfinance."""
      # Implementation...
  ```
- **Error Handling**: Use try-except blocks for external calls (e.g., yfinance). Log errors with Python's logging module.

## JavaScript (Alpine.js Frontend)

- **Components**: Keep components modular. Example:
  ```html
  <div x-data="{ stock: 'NIFTY50' }">
      <span x-text="stock"></span>
  </div>
  ```
- **State Management**: Use Alpine's x-data for local state; avoid global variables.
- **Asynchronicity**: Handle API calls with async/await for readability.

## Testing and Documentation

- **Testing**: Write unit tests for backend functions; use manual frontend tests per `docs/testing-strategy.md`.
- **Comments**: Explain non-obvious code; reference related docs (e.g., "See docs/api-reference.md for endpoint details").

For more, refer to `docs/architecture.md` and enforce via code reviews.