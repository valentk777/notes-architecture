Reality layer: distributed systems, DevOps, cloud-native, metrics, and governance.

---

## Distributed Systems Realities
- **Network unreliable, latency nonzero, bandwidth limited.**
- **Everything fails eventually.**
- **Consistency models:** Strong, Eventual, Causal.
- **Patterns:** Retry + Idempotency + Timeout + Circuit Breaker.
- **Observability:** logs + metrics + traces (OpenTelemetry).

---

## CI/CD & Governance
- Use **service templates** for consistent bootstraps (logging, health, tracing).
- Deploy small, safe, reversible increments.
- Store **ADRs** and **ARC42** per service.
- Automate compliance: lint, security scan, dependency update.
- Keep **ownership maps** clear (service → team).

---

## Metrics & Feedback Loops
### DORA Metrics
| Metric | Description | Target |
|--------|--------------|--------|
| Deployment frequency | How often code hits prod | multiple/day |
| Lead time | Commit → deploy | < 1 day |
| Change failure rate | % deployments failing | < 15% |
| Time to restore | Failure → recovery | < 1h |

**Measure via:**  
- GitHub Actions deploy count.  
- Merge→deploy timestamps.  
- Incident tags / rollbacks.  
- Monitoring recovery duration.

---

## Distributed Data Patterns
| Problem | Pattern | Notes |
|----------|----------|------|
| Cross-service consistency | Saga (compensating) | semantic undo, not rollback |
| Reliable publish | Outbox | DB + message atomicity |
| Schema evolution | Versioned contracts | backward compatible APIs |
| Shared contracts | Protos repo | regenerate clients automatically |

---

## Resilience & Scaling
- Use bulkheads, circuit breakers, retries with jitter.
- Apply autoscaling & backpressure.
- Cache at multiple levels (client, service, data).
- Monitor saturation (queue depth, latency percentiles).

---

## Quality Evaluation
- **ATAM (Architecture Trade-off Analysis Method):**
  - Identify business drivers → quality attributes.
  - Build *utility tree* → analyze risks, trade-offs, sensitivity points.
  - Output: risk list, non-risk list, sensitivity matrix.
- Run lightweight ATAM every major release.

---

## Observability Toolkit
- **Metrics:** latency (p95/p99), RPS, error rate, saturation.
- **Tracing:** distributed spans with correlation IDs.
- **Logs:** structured, JSON, PII-free.
- **Dashboards:** per service; alert thresholds = SLO + error budget.

---

## Governance Loop
1. Define rules (principles, metrics).
2. Measure (telemetry, DORA).
3. Review (ADR retro, ATAM workshop).
4. Adjust (template updates, tech radar).