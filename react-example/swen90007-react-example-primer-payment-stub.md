# Payment Stub

Your project also integrates with the **payment stub** - a stateless stand-in for a
3rd-party payment provider, published as a pre-built Docker image. Setup is identical to
the JSP track: follow [Step 10: Set Up the Payment Stub](../setup_dev/10_payment_stub.md)
for authenticating to GHCR (if needed), pulling and running the image (standalone or as a
service in your `docker-compose.yml`), the `/quote` and `/payments` endpoints, the
available configuration, and a Java client you can adapt for your API tier.

```{note}
You'll need Docker for this, set up in [Milestone 1: Deployment](swen90007-react-example-primer-milestone-1.md).
Otherwise this can be done at any point - it doesn't depend on the other milestones.
```

:::{admonition} React-specific notes
:class: tip
- **Call it from your API, not the browser.** The React SPA talks to your Servlet API; the
  Servlet API talks to the payment stub. Calling the stub directly from the browser would
  expose the `SHARED_SECRET` (if you set one) and put a third-party dependency directly in
  the client.
- **The UI needs a pending state.** Every stub response takes 3-8 seconds by design. Disable
  the submit control and show a spinner while the request is in flight, using the same
  Hooks-based async patterns covered in
  [Milestone 2: Core functionality](swen90007-react-example-primer-milestone-2.md).
- **Give `REJECTED` and a 5xx different UX.** A `REJECTED` response is a clean decline -
  show it as a normal validation-style message. A 5xx is a transient failure - show it as a
  retryable error, not a declined payment.
:::
