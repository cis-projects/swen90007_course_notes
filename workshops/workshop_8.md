# Workshop 8

```{admonition} By Now You Should Have
:class: important
- Submitted **Part 2** and attended your demonstration.
- Working instrumentation from [Workshop 7](workshop_7) — traces, a few spans, and export to a back-end.
- Begun **Part 3** — concurrency control, Unit of Work, and the performance exercise.
```

You load-test your application with **[k6](https://k6.io)**, a scripting-based load-testing tool. In this project it does two jobs, both assessed in Part 3:

1. it's how you show your **concurrency mechanisms behave correctly under genuine parallelism** — not just manual, UI-driven clicking; and
2. it drives an **evidence-based performance-improvement exercise**, where you use your telemetry to find a problem, fix it, and measure the before/after impact.

```{admonition} Run k6 locally
:class: warning
Load tests are run **locally**, never in CI (the spec forbids it) — and be careful pointing heavy load at Render, or you'll burn your deployment budget.
```

## A k6 script

A k6 test is a small JavaScript file describing what each **virtual user (VU)** does, how many run in parallel, and the pass/fail **thresholds**.

::::{admonition} `smoke.js` — a basic load test
:class: dropdown

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 50,             // 50 virtual users in parallel
  duration: '30s',
  thresholds: {
    http_req_failed:   ['rate<0.01'],   // < 1% of requests error
    http_req_duration: ['p(95)<500'],   // 95% of requests under 500 ms
  },
};

export default function () {
  const res = http.get('https://your-app.onrender.com/events');
  check(res, { 'status is 200': (r) => r.status === 200 });
}
```

Run it with `k6 run smoke.js`. Form-encoded Servlet endpoints are perfectly scriptable — you don't need a REST API for this.
::::

## Use 1 — concurrency under genuine parallelism

Your Part 3 concurrency testing **must not rely solely on the UI**. k6 lets many VUs hit the same operation at the same instant — the only honest way to prove a mechanism works. For the seat double-booking issue, for example, point many VUs at the *same* seat and assert that **exactly one** reservation succeeds and the seat is never sold twice.

You write one k6 strategy per concurrency issue, and present the outcome (text, tables, screenshots) in your Wiki (#6 Runtime View). You may be asked to run a script **live** during your demo.

## Use 2 — the performance-improvement exercise

This is a disciplined measurement loop, and it's the *discipline* that's assessed — not whether the fix was clever (or AI-assisted, which is allowed if disclosed).

| Step | What you do |
| --- | --- |
| **1. Baseline** | Run k6, record the numbers (latency, throughput, errors). |
| **2. Hypothesis** | Read your telemetry — e.g. a trace showing an **N+1** query in a listing/history use case, or an obvious caching opportunity. |
| **3. Change one thing** | Make a single, targeted improvement. |
| **4. Re-measure** | Run the *same* k6 test again. |
| **5. Explain** | Say *why* the change produced the measured effect. |

A typical before/after, fixing an N+1 in a booking-history listing:

| Metric | Before | After |
| --- | :---: | :---: |
| p95 latency | 1180 ms | 240 ms |
| Requests / second | 42 | 175 |
| DB queries per request | 51 (N+1) | 2 |

You can also use k6 to **support or disprove a hypothesis** — e.g. "if I add locking, I should see reliability improve at some cost to throughput." Trading one for the other, and showing it, is a strong result.

## Part 3 performance checklist

- [ ] **k6 scripts** committed under `load-tests/` in your repo.
- [ ] A k6 **concurrency strategy per concurrency issue**, driving genuine parallelism, with outcomes in your Wiki.
- [ ] A **before → after** improvement exercise: baseline, one change, re-measurement, and an explanation of why it worked.
- [ ] Any **GenAI assistance disclosed** in your Wiki.
- [ ] Your **golden-signals dashboards** (from [Workshop 7](workshop_7)) shown alongside the evidence.
