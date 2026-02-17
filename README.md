# AI Experts Assignment (Python)

# Bug Fix & Test Suite

A minimal Python HTTP client that handles OAuth2 token authentication automatically.
This project demonstrates how to identify, reproduce, and fix a subtle token refresh bug,
with full test coverage and Docker support for reproducible runs.

---

## The Bug (Summary)

The token refresh logic failed silently when `oauth2_token` was a plain dictionary
instead of a proper `OAuth2Token` object. The request would go out with no
`Authorization` header attached (only http_client.py is changed)

**Broken condition:**
```python
if not self.oauth2_token or (isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired):
```

**Fixed condition:**
```python
if not isinstance(self.oauth2_token, OAuth2Token) or self.oauth2_token.expired:
```

See `EXPLANATION.md` for the full breakdown.

---

## Project Structure
```
.
├── app/
│   ├── __init__.py
│   ├── http_client.py       # Client with OAuth2 token handling + thread lock
│   └── tokens.py            # OAuth2Token dataclass
├── tests/
│   ├── conftest.py
│   └── test_http_client.py  # Tests covering all token states 
├── Dockerfile
├── EXPLANATION.md
├── requirements.txt
└── README.md
```

---

## Running Tests Locally
```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run the test suite
PYTHONPATH=. python -m pytest -v --override-ini="addopts="
```

---

## Running Tests with Docker
```bash
# 1. Build the image
docker build -t oauth2-http-client .

# 2. Run the tests
docker run --rm oauth2-http-client
```

The container runs `python -m pytest -v` by default and exits with code `0` on success.

---

## What the Tests Cover

| Test | Scenario |
|---|---|
| `test_client_uses_requests_session` | Client initializes with a proper session |
| `test_token_from_iso_uses_dateutil` | Token is correctly parsed from ISO timestamp |
| `test_api_request_sets_auth_header_when_token_is_valid` | Valid token → header attached directly |
| `test_api_request_refreshes_when_token_is_missing` | `None` token → refresh triggered |
| `test_api_request_refreshes_when_token_is_dict` | Dict token → refresh triggered (the bug case) |

## Project structure

```
.
├── app/
│   ├── __init__.py
│   ├── http_client.py
│   └── tokens.py
├── tests/
│   ├── conftest.py
│   └── test_http_client.py
├── Dockerfile
├── EXPLANATION.md
├── requirements.txt
└── README.md
```

## Running tests locally

```bash
# Install dependencies
pip install -r requirements.txt

# Run the test suite
PYTHONPATH=. python -m pytest -v --override-ini="addopts="
# Or just use always
PYTHONPATH=. PYTEST_PLUGINS="" .venv/bin/python -m pytest -v --override-ini="addopts="
```

## Running tests with Docker

```bash
# Build the image
docker build -t ai-experts-assignment .

# Run the tests
docker run --rm ai-experts-assignment
```

The container runs `python -m pytest -v` by default and exits with code `0` on success.