# tap-amazon-seller
Forked from: https://gitlab.com/hotglue/tap-amazon-seller
`tap-amazon-seller` is a Singer tap for Amazon-Seller.

Built with the [Meltano Tap SDK](https://sdk.meltano.com) for Singer Taps.

## Installation

- [ ] `Developer TODO:` Update the below as needed to correctly describe the install procedure. For instance, if you do not have a PyPi repo, or if you want users to directly install from your git repo, you can modify this step as appropriate.

```bash
pipx install tap-amazon-seller
```

## Configuration

### Accepted Config Options

| Setting | Type | Required | Default | Description |
|---------|------|----------|---------|-------------|
| `lwa_client_id` | string | Yes | — | Login with Amazon (LWA) client ID for SP-API authentication |
| `client_secret` | string | Yes | — | LWA client secret |
| `refresh_token` | string | Yes | — | LWA OAuth refresh token |
| `aws_access_key` | string | No | — | AWS IAM access key (for role-based auth) |
| `aws_secret_key` | string | No | — | AWS IAM secret key (for role-based auth) |
| `role_arn` | string | No | — | AWS IAM role ARN to assume for SP-API access |
| `sandbox` | boolean | No | `false` | Use SP-API sandbox environment for testing |
| `marketplaces` | array/string | No | All 20 supported | List of marketplace IDs to sync (e.g. `["US", "CA"]`) |
| `report_types` | array/string | No | `["GET_LEDGER_DETAIL_VIEW_DATA", "GET_MERCHANT_LISTINGS_ALL_DATA"]` | SP-API report types to request |
| `processing_status` | array/string | No | `["IN_QUEUE", "IN_PROGRESS"]` | Report processing statuses to filter on |

### Known API Behavior

- **`financial_event_groups` stream:** The Amazon SP-API `listFinancialEventGroups` endpoint ignores the `FinancialEventGroupStartedAfter` date filter for Open settlement groups — they are always returned regardless of the date parameter. This means the stream may return groups older than the bookmark on incremental syncs. This is expected behavior; the data is merged/upserted downstream.
- **Start date:** When no bookmark exists, the tap falls back to the `start_date` config. The Amazon docs state that requesting data beyond 2 years should return an empty response ([ref](https://developer-docs.amazon.com/sp-api/reference/listfinancialeventsbygroupid)), but in practice the API throws a `SellingApiBadRequestException` instead. In the event that `SellingApiBadRequestException` is thrown, we catch it in `settlement_financial_events` stream and skip it.

### Source Authentication and Authorization

This tap authenticates via Amazon's Login with Amazon (LWA) OAuth flow. You need an SP-API application registered in Seller Central with the required API permissions. Provide `lwa_client_id`, `client_secret`, and `refresh_token`. Optionally use IAM role-based auth with `aws_access_key`, `aws_secret_key`, and `role_arn`.

## Usage

You can easily run `tap-amazon-seller` by itself or in a pipeline using [Meltano](https://meltano.com/).

### Executing the Tap Directly

```bash
tap-amazon-seller --version
tap-amazon-seller --help
tap-amazon-seller --config CONFIG --discover > ./catalog.json
```

## Developer Resources

- [ ] `Developer TODO:` As a first step, scan the entire project for the text "`TODO:`" and complete any recommended steps, deleting the "TODO" references once completed.

### Initialize your Development Environment

```bash
pipx install poetry
poetry install
```

### Create and Run Tests

Create tests within the `tap_amazon_seller/tests` subfolder and
  then run:

```bash
poetry run pytest
```

You can also test the `tap-amazon-seller` CLI interface directly using `poetry run`:

```bash
poetry run tap-amazon-seller --help
```

### Testing with [Meltano](https://www.meltano.com)

_**Note:** This tap will work in any Singer environment and does not require Meltano.
Examples here are for convenience and to streamline end-to-end orchestration scenarios._

Your project comes with a custom `meltano.yml` project file already created. Open the `meltano.yml` and follow any _"TODO"_ items listed in
the file.

Next, install Meltano (if you haven't already) and any needed plugins:

```bash
# Install meltano
pipx install meltano
# Initialize meltano within this directory
cd tap-amazon-seller
meltano install
```

Now you can test and orchestrate using Meltano:

```bash
# Test invocation:
meltano invoke tap-amazon-seller --version
# OR run a test `elt` pipeline:
meltano elt tap-amazon-seller target-jsonl
```

### SDK Dev Guide

See the [dev guide](https://sdk.meltano.com/en/latest/dev_guide.html) for more instructions on how to use the SDK to 
develop your own taps and targets.
