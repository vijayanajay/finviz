# Environment Variables for Visual Market Pulse Dashboard

## Overview

This document details the environment variables used in the project, focusing on sensitive configurations for the backend and deployment. These are managed in a `.env` file for security.

## Key Environment Variables

- **FLASK_ENV**: Set to 'development' or 'production' to control app behavior (e.g., debug mode).
- **YFINANCE_API_KEY**: Optional API key for yfinance if required (though yfinance doesn't typically need one).
- **CACHE_TYPE**: Specifies the caching backend, e.g., 'redis' or 'filesystem'.
- **REDIS_URL**: URL for Redis instance if used for caching (e.g., 'redis://localhost:6379').
- **SECRET_KEY**: A random string for securing sessions in Flask.

## Best Practices
- Store in `.env` and add to `.gitignore` to avoid committing sensitive data.
- Reference in code via `os.environ.get('VAR_NAME')`.
- For production on Render, set via platform environment variables.

See `docs/architecture.md` for usage context.