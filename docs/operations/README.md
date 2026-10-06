# Orbit Testing: operations and security

This guide organizes runtime behavior documented by the package. It does not certify production readiness. Verify provider/client versions, permissions, transport security, limits, and failure behavior in the target environment before release.

## Configuration surface

Environment names found in the package README:

The package README does not name `ORBIT_*` variables. Use its typed constructors and application configuration, and confirm exact runtime inputs in the implementation before deployment.

Use the package README's constructor and deployment examples. Store credentials in a secret manager and avoid logging credentials, raw provider errors, request data, or opaque cursors.

## Lifecycle, failure behavior, and limits

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

## Status

This package is pre-alpha; its API is not stable and it is not yet a published release. It supports
Python 3.11 through 3.14.

## Production validation

Validate startup/shutdown cleanup, timeout and cancellation behavior, concurrency and payload bounds where applicable, secret rotation and least-privilege access, data durability, backup/restore, and failover against the selected provider. Do not infer distributed or durable guarantees from an in-process API or fake-client tests.
