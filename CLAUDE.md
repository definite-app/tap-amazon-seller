# CLAUDE.md - tap-amazon-seller

## Project Overview
Singer tap for Amazon Seller Central built with Meltano Singer SDK. Extracts data from Amazon SP-API (orders, inventory, reports, vendor data, etc.) across 20 streams.

## Tech Stack
- **Python** 3.7.1 - <3.11 (primary: 3.8-3.9)
- **singer-sdk** ^0.4.4 (Meltano SDK)
- **python-amazon-sp-api** 1.6.93
- **Package manager:** Poetry

## Key Commands

```bash
# Install dependencies
poetry install

# Run the tap
poetry run tap-amazon-seller --config CONFIG --discover > ./catalog.json

# Tests
poetry run pytest
tox -e pytest          # Full test matrix (3.7, 3.8, 3.9)

# Formatting & Linting
tox -e format          # Auto-format (black + isort)
tox -e lint            # Lint check (flake8, mypy, pydocstyle, black --check, isort --check)
poetry run black tap_amazon_seller/
poetry run flake8 tap_amazon_seller
poetry run mypy tap_amazon_seller
poetry run isort tap_amazon_seller
```

## Project Structure

```
tap_amazon_seller/
├── tap.py          # TapAmazonSeller class, config schema, stream discovery
├── client.py       # Base AmazonSellerStream class, SP-API client helpers, retry logic
├── streams.py      # 20 stream class definitions (orders, inventory, reports, vendor, etc.)
├── reportsv3.py    # ReportsV3 API wrapper
├── amz_objects.py  # CatalogItems v2 API extension
├── utils.py        # Timeout/InvalidResponse exceptions, @timeout decorator
└── tests/
    └── test_core.py
```

## Key Patterns
- Parent-child streams: most streams are children of `MarketplacesStream`
- Exponential backoff retries (base factor 3-5, max 10 retries) for rate limiting
- Report streams use async polling: create report → poll for completion → download
- `SellingApiForbiddenException` is caught and streams are skipped gracefully
- State management uses bookmark partitions for incremental replication

## Code Style
- Black formatting (88 char line limit)
- isort for import ordering
- flake8 linting (max complexity 10)
- mypy type checking (Python 3.9 target)
- pydocstyle docstring validation
