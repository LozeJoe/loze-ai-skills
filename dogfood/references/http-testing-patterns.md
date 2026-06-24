# HTTP-Level Web App Testing Patterns

Reusable Python snippets for systematic QA testing without a browser.

## Route Enumeration & Status Check

```python
import urllib.request, urllib.error

base = 'http://localhost:PORT'
routes = [
    ('GET', '/'),
    ('GET', '/detail?id=1'),
    ('GET', '/detail?id=999'),   # non-existent
    ('GET', '/detail?id=-1'),     # negative
    ('GET', '/detail?id=abc'),    # non-numeric
    ('POST', '/login'),
]

for method, path in routes:
    try:
        req = urllib.request.Request(f'{base}{path}', method=method)
        r = urllib.request.urlopen(req, timeout=5)
        html = r.read().decode('utf-8', errors='replace')
        title = ''
        if '<title>' in html:
            s = html.index('<title>') + 7
            e = html.index('</title>', s)
            title = html[s:e]
        icon = '⚠️' if r.status >= 400 else ('🔸' if len(html) < 500 else '✅')
        print(f"{icon} {method} {path} -> {r.status}, {len(html)}B, '{title}'")
    except urllib.error.HTTPError as e:
        print(f"❌ {method} {path} -> HTTP {e.code}")
    except Exception as e:
        print(f"❌ {method} {path} -> {e}")
```

## Session & Login Flow

```python
import http.cookiejar

cj = http.cookiejar.CookieJar()
opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))

# Login
data = urllib.parse.urlencode({'username': 'u', 'password': 'p'}).encode()
req = urllib.request.Request(f'{base}/login', data=data, method='POST')
req.add_header('Content-Type', 'application/x-www-form-urlencoded')
r = opener.open(req, timeout=5)

# Check session
r = opener.open(f'{base}/profile', timeout=5)
```

## Security Header Audit

```python
security_headers = [
    'X-Frame-Options', 'X-Content-Type-Options',
    'Content-Security-Policy', 'X-XSS-Protection',
    'Strict-Transport-Security'
]
for h in security_headers:
    if h in r.headers:
        print(f"  {h}: ✅ {r.headers[h]}")
    else:
        print(f"  {h}: ❌ MISSING")
```

## CSRF Check

```python
html = r.read().decode('utf-8', errors='replace')
has_csrf = any(t in html.lower() for t in ['_csrf', 'token', 'nonce'])
print(f"CSRF: {'Present' if has_csrf else '⚠️ ABSENT'}")
```

## Input Fuzzing

```python
payloads = [
    ('empty', {'field': '', 'total': ''}),
    ('negative', {'field': 'x', 'total': '-100'}),
    ('SQLi', {'field': "';DROP TABLE--", 'total': '1'}),
    ('XSS', {'field': '<script>alert(1)</script>', 'total': '1'}),
]
for name, data in payloads:
    body = urllib.parse.urlencode(data).encode()
    req = urllib.request.Request(f'{base}/endpoint', data=body, method='POST')
    req.add_header('Content-Type', 'application/x-www-form-urlencoded')
    r = opener.open(req, timeout=5)
    print(f"  {name}: HTTP {r.status}, {len(r.read())}B")
```

## Link/Resource Audit

```python
import re
links = re.findall(r'href="([^"]+)"', html)
for link in set(links):
    try:
        r = urllib.request.urlopen(f'{base}{link}', timeout=5)
        print(f"  ✅ {link} ({r.status})")
    except:
        print(f"  ❌ {link}")
```

## Key Indicators Per Bug Class

| Bug Class | What to Check |
|-----------|---------------|
| Auth redirect | Logged-in user hitting /login should redirect home, not show form |
| 404 correctness | Non-existent resources must return 404 (not 200 with error page) |
| Forbidden vs 404 | Restricted pages should return 403 (not 404) for auth'd users |
| GET mutability | addOne/subOne/delete/addToCart must use POST (not GET) |
| Client trust | Payment total should be server-calculated, not from hidden input |
| URL consistency | logout vs loginout, myOrder vs orderList — pick one |
| Dead-end UX | Login/register pages without navbar trap users |
| Quantity validation | Must reject 0, -1, 999999, non-numeric |
