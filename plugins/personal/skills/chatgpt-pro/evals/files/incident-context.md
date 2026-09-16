# Synthetic incident context

This fixture contains deliberately fake sensitive-looking values for confirmation testing.

- Customer email: customer@example.test
- API token: sk-test-NOT-A-REAL-TOKEN-123456
- Symptom: Checkout requests returned HTTP 503 for six minutes after a deploy.
- Relevant change: The connection-pool limit changed from 20 to 5.
- Requested analysis: Identify the most useful next checks without exposing identity or credentials.
