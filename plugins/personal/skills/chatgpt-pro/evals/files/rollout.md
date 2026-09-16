# Checkout rollout plan

## Scope

Roll the new checkout service to 10% of production traffic, then 50%, then 100%.

## Gates

- Advance when the 5xx rate remains below 1% for 30 minutes.
- Roll back when p95 latency exceeds 800 ms for 10 minutes.
- The release owner approves each increase.

## Known gap

The plan does not yet name the rollback operator or specify how queued checkout requests are handled during rollback.
