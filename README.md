# Bocadillo

[![travis](https://img.shields.io/travis-ci/florimondmanca/bocadillo.svg)][travis-url]
[![python](https://img.shields.io/pypi/pyversions/bocadillo.svg)][pypi-url]
[![pypi](https://img.shields.io/pypi/v/bocadillo.svg)][pypi-url]
[![license](https://img.shields.io/pypi/l/bocadillo.svg)][pypi-url]

Inspired by [Responder](http://python-responder.org), Bocadillo is a web framework that combines ideas from Falcon and Flask while leveraging modern Python async capabilities.

Under the hood, it uses the [Starlette](https://www.starlette.io) ASGI toolkit and the [uvicorn](https://www.uvicorn.org) ASGI server.

## Contents

- [Quick start](#quick-start)
- [Install](#install)
- [Features](#features)
- [Contributing](#contributing)
- [Changelog](#changelog)
- [Roadmap](#roadmap)

## Quick start

Write your first app:

```python
# app.py
import bocadillo

api = bocadillo.API()

@api.route('/add/{x:d}/{y:d}')
async def add(req, resp, x: int, y: int):
    resp.media = {'result': x + y}

if __name__ == '__main__':
    api.run()
```

Run it:

```bash
python app.py
# or directly using uvicorn:
uvicorn app:api
```

```
INFO: Started server process [81910]
INFO: Waiting for application startup.
INFO: Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

Make some requests!

```bash
curl http://localhost:8000/add/3/5
{"result": 5}
```

🌯💥

## Install

Bocadillo is available on PyPI:

```bash
pip install bocadillo
```

## Usage

It all starts with an import:

```python
import bocadillo
api = bocadillo.API()
```

### Routing

#### Basics

To register a new route, use the `@api.route()` decorator:

```python
@api.route('/posts/{slug}')
def retrieve_post(req, res, slug: str):
    res.content = 'My awesome post'
```

Routes can also be declared in an `async` fashion, which allows you to call
arbitrary async/await Python code:

```python
from asyncio import sleep

async def find_post_content(slug: str):
    await sleep(1)  # perhaps query a database here?
    return 'My awesome post'

@api.route('/posts/{slug}')
async def retrieve_post(req, res, slug: str):
    res.content = await find_post_content(slug)
```

#### Restricting available HTTP methods

You can use the `methods` argument to `@api.route()` to restrict the set of
HTTP methods exposed on a route:

```python
@api.route('/posts', methods=['post'])
def create_post(req, res):
    # process `req`…
    res.status_code = 201
```

#### Class-based views

Additionally to the function-based, Bocadillo supports class-based views.
Each HTTP method gets mapped to the corresponding lower-cased method on the
class, e.g. `GET` maps to `.get()`.

```python
@api.route('/')
class Index:

    def get(self, req, res):
        res.content = 'Bocadillo'
```

Also, a catch-all `.handle()` method can be
implemented to process all requests — other methods will then be ignored.

```
```

### Sending responses

Bocadillo handles the nitty gritty of build HTTP responses for you. All you
have to do is set `.content` (for plain text), `.media` (for JSON) or `.html`
(for HTML) on the `Response` object in a view. Bocadillo takes charge of
serializing and setting the `Content-Type` header.

```python
@api.route('/multiply/{x}/{y}')
def joke(req, res, x, y):
    res.media = {'result': x + y}
```

```python
@api.route('/')
def index(req, res):
    res.html = '<h1>Hello, Bocadillo!</h1>'
```

You can set the status code on the response. Bocadillo does not provide
an enum of HTTP status codes. For now, you'll be safe using the
`http.HTTPStatus` enum from the standard library:

```python
from http import HTTPStatus

@api.route('/jobs', methods=['post'])
def create_job(req, res):
    # process `req`…
    res.status_code = 201
    # or:
    res.status_code = HTTPStatus.CREATED.value
```

### Error handling

You can raise an `HTTPError` exception in any view to trigger an appropriate
automatic error response:

```python
from bocadillo.exceptions import HTTPError

@api.route('/fail/{status_code:d}')
def fail(req, res, status_code: int):
    raise HTTPError(status_code)
```

```bash
curl -SD - http://localhost:8000/fail/401
```

```http
HTTP/1.1 401 Unauthorized
server: uvicorn
date: Wed, 07 Nov 2018 19:55:56 GMT
content-type: text/plain
transfer-encoding: chunked

Unauthorized
```

You can also register your own error handlers using `@api.error_handler()`.

```python
@api.error_handler(KeyError)
def on_key_error(req, res, exc: KeyError):
    res.status = 400
    res.content = f"You fool! We didn't find the key '{exc.args[0]}'."
```

For convenience, a non-decorator syntax is also available:

```python
def on_attribute_error(req, res, exc: AttributeError):
    res.status = 500
    res.media = {'error': {'attribute_not_found': exc.args[0]}}

api.add_error_handler(AttributeError, on_attribute_error)
```

## Features

- ASGI-compatible app
- Flask-inspired decorator-based routing
- Formatted string route patterns
- Falcon-inspired passing of request and response
- Send JSON responses using `resp.media`
- Class-based views
- Response headers
- Status codes
- HTTP error exceptions
- Jinja2 template rendering
- Static assets
- Mount any WSGI or ASGI app as a sub-app.

## Contributing

See [CONTRIBUTING](https://github.com/florimondmanca/bocadillo/blob/master/CONTRIBUTING.md) for contribution guidelines.

## Changelog

See [CHANGELOG](https://github.com/florimondmanca/bocadillo/blob/master/CHANGELOG.md) for a chronological log of changes to Bocadillo.

## Roadmap

If you are interested in the future features that may be implemented into Bocadillo, take a look at our [milestones](https://github.com/florimondmanca/bocadillo/milestones?with_issues=no).

To see what has already been implemented for the next release, see the [Unreleased](https://github.com/florimondmanca/bocadillo/blob/master/CHANGELOG.md#unreleased) section of our changelog.

<!-- URLs -->

[travis-url]: https://travis-ci.org/florimondmanca/bocadillo

[pypi-url]: https://pypi.org/project/bocadillo/
