
#networking #http #https #devops

---

## What is HTTP?

HyperText Transfer Protocol — the application-layer protocol browsers and servers use to exchange web content. **Request-response** based, runs on top of **TCP**.

```
Browser -> GET request ->  Server (port 80)
Browser <- HTML response <-  Server
```

Every HTTP request first does a TCP 3-way handshake, then sends the HTTP data.

```bash
curl -v http://example.com          # -v shows full request/response exchange
```

## Anatomy of an HTTP Request

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: curl/8.1.2
Accept: */*
```

- **Method** — action (`GET`, `POST`, etc.)
- **Path** — resource (`/index.html`)
- **Version** — `HTTP/1.1`, `HTTP/2`, etc.
- **Headers** — metadata (key: value)
- **Body** — optional, used in `POST`/`PUT`

```bash
curl -v http://example.com 2>&1 | grep "^>"     # request lines only
```

## Anatomy of an HTTP Response

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
Date: Sun, 11 Oct 2026 06:30:00 GMT

<html>...</html>
```

- **Status line** — version + status code + reason phrase
- **Headers** — metadata about response
- **Body** — actual content (HTML, JSON, image, etc.)

```bash
curl -v http://example.com 2>&1 | grep "^<"     # response lines only
```

## HTTP Methods

|Method|Purpose|
|---|---|
|`GET`|Retrieve data, no side effects|
|`POST`|Submit data, often creates something|
|`PUT`|Replace a resource entirely|
|`PATCH`|Partially update a resource|
|`DELETE`|Remove a resource|
|`HEAD`|Like GET, headers only, no body|
|`OPTIONS`|Ask what methods/actions are allowed|

```bash
curl -X POST -d "name=test" http://example.com/api        # POST with data
curl -I http://example.com                                   # HEAD request
curl -X OPTIONS http://example.com                              # allowed methods
```

## Status Codes

|Range|Category|Examples|
|---|---|---|
|1xx|Informational|`100 Continue`|
|2xx|Success|`200 OK`, `201 Created`, `204 No Content`|
|3xx|Redirection|`301 Moved Permanently`, `302 Found`, `304 Not Modified`|
|4xx|Client error|`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`|
|5xx|Server error|`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`|

```bash
curl -o /dev/null -s -w "%{http_code}\n" http://example.com/nonexistent-page     # status code only
```

## Headers Worth Knowing

**Request headers:**

```
Host: example.com                  # which site (one IP can host many domains)
User-Agent: ...                      # client making the request
Authorization: Bearer <token>          # credentials
Cookie: session=abc123                   # stored session data
Content-Type: application/json             # format of request body
```

**Response headers:**

```
Content-Type: text/html             # format of response body
Set-Cookie: session=abc123             # tells browser to store a cookie
Cache-Control: max-age=3600               # how long to cache response
Location: https://example.com/new           # redirect target (used with 3xx)
```

## HTTP is Stateless — Cookies Fix That

Each request is independent; server doesn't inherently remember you. **Cookies** maintain sessions — server sets one, browser sends it back on every subsequent request.

```bash
curl -c cookies.txt http://example.com        # save cookies to a file
curl -b cookies.txt http://example.com          # send cookies back
```

## HTTPS — HTTP + Encryption

HTTP wrapped inside **TLS** (Transport Layer Security, successor to SSL). Runs on **port 443**.

```
HTTP:  Browser <-> Server   (plain text, readable on the network)
HTTPS: Browser <-> [TLS encrypted tunnel] <-> Server
```

TLS provides:

- **Encryption** — data unreadable in transit
- **Integrity** — tampering is detectable
- **Authentication** — verifies you're talking to the real server (via certificates)

```bash
curl -v https://example.com 2>&1 | grep -i "SSL\|TLS"      # see TLS handshake details
```

## The TLS Handshake (Simplified)

```
1. Client -> ClientHello -> Server      "Here's what TLS versions/ciphers I support"
2. Client <- ServerHello <- Server       "Here's what we'll use, and my certificate"
3. Client verifies the certificate         (signed by trusted CA, matches domain)
4. Client & Server derive a shared encryption key
5. Encrypted application data (HTTP) now flows
```

Extra round trips vs plain HTTP (though TLS 1.3 reduced this significantly).

## Certificates — How Trust Works

A certificate proves server identity, issued by a **Certificate Authority (CA)** your browser/OS already trusts.

```bash
openssl s_client -connect example.com:443 -showcerts        # full certificate chain
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates    # validity dates
```

Chain: **Root CA → Intermediate CA → Your server's certificate**. Browser trusts Root, which vouches for Intermediate, which vouches for the server.

## Invalid Certificate Scenarios

|Problem|Browser shows|
|---|---|
|Certificate expired|"Your connection is not private"|
|Domain doesn't match certificate|"Certificate is not valid for this site"|
|Self-signed / untrusted CA|"Certificate not trusted"|
|Certificate revoked|"Certificate has been revoked"|

```bash
curl https://expired.badssl.com        # deliberately broken cert, will fail
curl -k https://expired.badssl.com       # -k skips verification (never in production)
```

## HTTP/1.1 vs HTTP/2 vs HTTP/3

- **HTTP/1.1** — one request per connection at a time (unless rarely-used pipelining); text-based
- **HTTP/2** — multiplexes many requests over one connection, binary framing, faster for asset-heavy pages
- **HTTP/3** — runs over **QUIC** (built on UDP, not TCP!), avoids TCP's head-of-line blocking

```bash
curl -v --http2 https://example.com 2>&1 | grep "HTTP/2"     # check HTTP/2 support
```

> HTTP/3 is a notable case of a traditionally TCP-based protocol moving to a UDP foundation (with its own reliability layer via QUIC) for performance.

## Redirects in Action

```bash
curl -v http://example.com          # many sites auto-redirect HTTP -> HTTPS (301/302)
curl -L http://example.com            # -L follows redirects automatically
```

---

## Practice Problems

- [ ] `curl -v http://example.com` — identify TCP handshake, request, and response separately
- [ ] `curl -I http://example.com` — confirm headers only, no body
- [ ] `curl -o /dev/null -s -w "%{http_code}\n" http://example.com/doesnotexist` — confirm `404`
- [ ] `curl -v https://github.com 2>&1 | grep -i "set-cookie\|cache-control"` — find session/cache headers
- [ ] `curl -v https://example.com 2>&1 | grep -i "SSL\|TLS"` — note TLS version used
- [ ] `echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates` — check cert expiry
- [ ] `curl https://expired.badssl.com` — see an invalid-certificate error
- [ ] `curl -v http://github.com` — find the redirect to HTTPS

