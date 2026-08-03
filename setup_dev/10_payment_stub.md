# Step 10: Set Up the Payment Stub

Your project integrates with a **payment stub** - a stateless stand-in for a 3rd-party
payment provider. It stores nothing: it receives a request, applies a deterministic
decline rule, injects a realistic network delay (and, optionally, a random failure), and
returns. It exists to give you a taste of working with an external service that your
application does not control - including the parts of that experience that are
inconvenient by design, like latency and occasional failures.

The payment stub runs as its own container, alongside your application, using the same
Docker skills you picked up in [Step 8](8_setup_docker.md). It is published as a
pre-built image, so you do not need its source code - just Docker.

```{note}
Should be done by all team members.
```

## Prerequisites

```{note}
Docker Desktop must be installed and running. If you have not done this yet, go back to
[Step 8: Setup Docker](8_setup_docker.md) first.
```

## Authenticate to GHCR

```{note}
The payment stub image is currently **public** - you can skip this step and go straight
to `docker pull` below. This section is kept here in case visibility changes back to
private later.
```

The payment stub image is published to GitHub Container Registry (GHCR). If the image is
private, log in once with a GitHub Personal Access Token (PAT) before you can pull it:

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens
   (classic) → Generate new token.
2. Scope: `read:packages` only.
3. Log in from your terminal:

```
echo "<YOUR_PAT>" | docker login ghcr.io -u <YOUR_GITHUB_USERNAME> --password-stdin
```

You only need to do this once per machine - the login is cached by Docker.

## Pull the Image

```
docker pull ghcr.io/swen90007-2026/payment-stub:latest
```

Other available tags:
- `latest` - most recent build of `main`
- `sha-<short-sha>` - a specific commit
- `vX.Y.Z` - a specific release, if tagged

## Run It

Quickest way, with defaults (payment threshold $1000, 10% processing fee, 3-8s simulated
latency, no random failures):

```
docker run -p 8080:8080 ghcr.io/swen90007-2026/payment-stub:latest
```

Check it is alive:

```
curl http://localhost:8080/healthz
```

### Run with Docker Compose

If your application already has a `docker-compose.yml`, it is easier to add the payment
stub as another service in the same file, rather than running it separately:

```yaml
services:
  payment-stub:
    image: ghcr.io/swen90007-2026/payment-stub:latest
    ports:
      - "8080:8080"
    environment:
      DECLINE_THRESHOLD: "1000.00"
      PROCESSING_FEE_RATE: "0.10"
      LATENCY_MIN_MS: "3000"
      LATENCY_MAX_MS: "8000"
      FAILURE_RATE: "0.0"
```

```{tip}
Add this as a service alongside your own app's compose file so both run together on the
same Docker network, and your application can reach the stub at `http://payment-stub:8080`.
```

## The Endpoints

### `POST /quote`

Call this the moment your app attempts to use the payment service (for example, when a
user lands on the payment step). Returns a processing fee for the amount, plus a decision
that mimics a rejection when the amount is too large.

```
curl -s http://localhost:8080/quote \
  -H 'Content-Type: application/json' \
  -d '{"amount": 120.00, "attendeeRef": "attendee-123"}'
```

```
{
  "quoteRef": "b1e2c3d4-...",
  "processingFee": 12.00,
  "status": "ACCEPTED",
  "reason": null,
  "timestamp": "2026-07-19T10:15:30.000Z"
}
```

### `POST /payments`

Call this when a user actually pays for a booking.

```
curl -s http://localhost:8080/payments \
  -H 'Content-Type: application/json' \
  -d '{"amount": 120.00, "bookingRef": "booking-456"}'
```

Both endpoints apply the same rule: an amount above `DECLINE_THRESHOLD` returns `REJECTED`
with reason `AMOUNT_ABOVE_THRESHOLD`. Every response is delayed by a random amount between
`LATENCY_MIN_MS` and `LATENCY_MAX_MS` (default: at least ~3 seconds), and with probability
`FAILURE_RATE` the request instead fails with a configured 5xx - a separate, transient
failure path distinct from a clean `REJECTED` decision.

### `GET /healthz`

Liveness check. Not subject to latency, failure injection, or auth.

## Configuration

All configuration is via environment variables, readable at container runtime - no image
rebuild required.

| Variable | Default | Meaning |
|---|---|---|
| `PORT` | `8080` | HTTP listen port |
| `DECLINE_THRESHOLD` | `1000.00` | Amounts strictly above this are `REJECTED` |
| `PROCESSING_FEE_RATE` | `0.10` | Fraction of amount returned as `processingFee` by `/quote` |
| `LATENCY_MIN_MS` | `3000` | Minimum artificial delay applied to every response |
| `LATENCY_MAX_MS` | `8000` | Maximum artificial delay (must be ≥ `LATENCY_MIN_MS`) |
| `FAILURE_RATE` | `0.0` | Probability (0-1) of an injected 5xx failure |
| `FAILURE_HTTP_STATUS` | `503` | HTTP status used for the injected failure |
| `FAILURE_CODE` | `UPSTREAM_UNAVAILABLE` | Machine-readable code in the injected failure body |
| `FAILURE_MESSAGE` | `Payment provider temporarily unavailable` | Human-readable message in the injected failure body |
| `SHARED_SECRET` | (unset) | If set, requests must send a matching `X-Payment-Secret` header, or receive `401` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | (unset) | OTLP/HTTP collector endpoint. Telemetry is fully disabled while unset |
| `OTEL_EXPORTER_OTLP_HEADERS` | (unset) | OTLP auth headers, e.g. `Authorization=Basic <token>` for Grafana Cloud |
| `OTEL_SERVICE_NAME` | `payment-stub` | `service.name` on emitted spans/metrics |

For example, to exercise your application's error-handling path, force every request to
fail:

```
docker run -p 8080:8080 -e FAILURE_RATE=1.0 ghcr.io/swen90007-2026/payment-stub:latest
```

## Observability (optional)

The stub is instrumented with OpenTelemetry (traces + metrics), but export is **off by
default** - with no telemetry config set, it just runs standalone with no errors. If you
want to see requests flowing through it as traces/metrics (for example, to demonstrate
distributed tracing across your app and the stub), set `OTEL_EXPORTER_OTLP_ENDPOINT` (and
`OTEL_EXPORTER_OTLP_HEADERS` if your collector needs auth) the same way as any other
config - via `-e` flags on `docker run`, or `environment:` keys in your compose service.

```{tip}
See the payment-stub project's README "Observability / OpenTelemetry" section for the
exact setup of both a hosted Grafana Cloud endpoint and a local `otel-lgtm` stack you can
run entirely on your own machine.
```

## Troubleshooting

- **`unauthorized` / `denied` on pull** - re-run the `docker login ghcr.io` step; PATs can
  expire.
- **Nothing responds on `localhost:8080`** - check `docker ps`; ensure nothing else is
  bound to port 8080.
- **Every request takes at least 3 seconds** - expected; the stub simulates latency
  (`LATENCY_MIN_MS=3000` by default).

```{admonition} What's Next
This is the final step of the project setup guide. You should now have your project
running locally with IntelliJ, Tomcat, PostgreSQL, Docker, a deployment on Render, and the
payment stub running alongside your application. Head back to the workshops or your
project brief to start integrating the payment stub into your application.
```
