# WebHawk

A Python web vulnerability scanner. It checks a website for basic weaknesses: missing security headers, SSL cert expiry, SQL error reflection, and open web ports. It saves the full report to `web_report.json`.

Built to learn real web checks: headers, SSL with socket, error reflection, and port scanning.

## What It Does

1. Checks for missing security headers
2. Checks SSL cert expiry and days left
3. Checks for SQL error reflection with a test payload
4. Checks web ports 80, 443, 8080, 8443
5. Saves all findings to `web_report.json`

## Functions

### `check_headers(url)`
- Input: `url` - string, e.g. `"https://example.com"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Uses `requests.get` with timeout 5
- Checks for:
    - `Content-Security-Policy`
    - `X-Frame-Options`
    - `X-Content-Type-Options`
    - `Strict-Transport-Security`
- If missing: `severity` MEDIUM, `message` MISSING
- If present: `severity` OK, `message` OK
- If request fails: makes error finding and returns

### `check_ssl(url)`
- Input: `url` - string, e.g. `"https://example.com"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Gets host with `urlparse`, port is 443 for https
- Connects with `socket.create_connection` and wraps with `ssl.create_default_context`
- Gets cert with `getpeercert()`, parses `notAfter` with `datetime.strptime`
- Calculates days left from today to expiry
- If days left under 30: `severity` HIGH
- If days left 30 or more: `severity` OK
- Uses timezone-aware dates to avoid `DeprecationWarning`

### `check_sqli_reflection(url)`
- Input: `url` - string, e.g. `"https://example.com"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Builds test url with payload `?q=' OR 1=1 --`
- Gets test url with `requests.get`
- Checks response text, lowercased, for SQL error strings like `sql syntax`, `mysql`, `you have an error in your sql syntax`, `unclosed quotation mark`
- If found: `severity` HIGH, `message` Possible SQL error reflection
- If not found: `severity` OK, `message` No SQL error reflection
- This only checks for error reflection, it does not exploit

### `check_open_ports_web(url)`
- Input: `url` - string, e.g. `"https://example.com"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Gets host with `urlparse`
- For each port in `80, 443, 8080, 8443`, tries `socket.connect_ex`
- If `connect_ex` returns 0: port is OPEN, `severity` INFO
- If it returns non-zero: port is CLOSED, `severity` OK
- Closes socket after each port

### `main()`
- Takes URL from `sys.argv`
- Calls all 4 check functions
- Combines all lists into `all_findings`
- Saves to `web_report.json` with `json.dump` and `indent=4`
- Prints each finding as `[SEVERITY] check: message`

## Quick Start

```bash
git clone https://github.com/YOUR-USERNAME/WebHawk.git
cd WebHawk
pip install -r requirements.txt
