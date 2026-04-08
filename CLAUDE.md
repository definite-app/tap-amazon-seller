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

## Known SP-API Quirks
- `listFinancialEventGroups`: The `FinancialEventGroupStartedAfter` date filter only applies to Closed groups. Open groups are always returned regardless of the date filter. This means the `financial_event_groups` stream may return groups older than the bookmark on every sync, which is expected — data is merged downstream.
- `listFinancialEventsByGroupId`: The API enforces a ~2-year data retention period. Requesting events for a group older than ~2 years throws a `SellingApiBadRequestException`. The `settlement_financial_events` child stream skips groups older than 2 years before making the API call, using `FinancialEventGroupStart` from the parent context.
- `listFinancialEventGroups`: We have noticed on some accounts that the date filter is ignored for certain Closed groups as well, returning them regardless of the filter value.

## Incremental Sync / start_date
- When no bookmark exists (first sync or full refresh), the Singer SDK falls back to the `start_date` config from `meltano.yml`. If `start_date` is also not set, streams default to their own hardcoded fallback.
- For `financial_event_groups`, the start date is clamped to 729 days ago (just under 2 years) regardless of the `start_date` config value or bookmark, since the API has a ~2-year retention limit and the child `settlement_financial_events` stream cannot retrieve events for older groups. The API always returns Open groups regardless of date filter, so old Open groups may be passed to the child stream — they are skipped if older than 2 years.

## Code Style
- Black formatting (88 char line limit)
- isort for import ordering
- flake8 linting (max complexity 10)
- mypy type checking (Python 3.9 target)
- pydocstyle docstring validation
