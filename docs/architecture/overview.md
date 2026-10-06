# Orbit Testing: architecture and boundaries

## Responsibility

`orbit-testing` provides an in-process client for testing Orbit applications through the real
ASGI protocol, without opening sockets or installing another web framework.

```bash
pip install orbit-core orbit-testing
```

```python
from orbit import Application, ApplicationConfig
from orbit.asgi import ASGIApplication, Response
from orbit_testing import TestClient

application = Application(ApplicationConfig(name="example"))


@application.router.route("/", name="home")
async def home(request):
    return Response.text("hello")


async def test_home():
    async with TestClient(ASGIApplication(application)) as client:
        response = await client.request("GET", "/")
    assert response.status == 200
    assert response.body == b"hello"
```

The client sends ASGI lifespan startup and shutdown messages, bounds request path/query/header
inputs using Core's public HTTP limits, preserves repeated response headers, and rejects malformed
response framing. Captured responses default to a 16 MiB body limit and 65,536 frames; applications
can lower or raise these within the hard 1 GiB and 1,000,000-frame bounds. Its requests are fully
buffered and it models a client disconnect after the request body; it is not a browser, socket
client, load generator, or substitute for host-level integration tests.

## Declared dependencies

The following dependency declarations come from the checked-in manifests. Optional groups and development dependencies are called out separately.

### `pyproject.toml`
- `orbit-core>=0.1.0a1,<0.2`
- Optional `dev` group: `pytest>=8,<10`, `pytest-asyncio>=0.24,<2`, `ruff>=0.8,<1`, `mypy>=1.13,<2`.

Declared dependencies do not mean that optional providers or services are bundled with this package.

## Implementation layout

Representative implementation files in this checkout:

- `src/orbit_testing/__init__.py`
- `src/orbit_testing/client.py`

## Public contract and scope

## Status

This package is pre-alpha; its API is not stable and it is not yet a published release. It supports
Python 3.11 through 3.14.

## Boundary rules

Keep provider SDKs, credentials, transports, and provider-specific error translation in provider adapters. Keep reusable capability contracts in the matching capability package and lifecycle orchestration in Core. Apply the relevant layer for this repository and preserve the dependency direction shown above.
