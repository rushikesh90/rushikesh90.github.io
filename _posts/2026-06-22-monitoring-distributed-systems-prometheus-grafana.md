---
layout: single
title: "Monitoring Distributed Systems with Prometheus and Grafana: A Practical Guide for Go Developers"
date: 2026-06-22
categories: [monitoring, distributed-systems, prometheus, grafana]
tags: [prometheus, grafana, monitoring, go, metrics, observability, distributed-systems]
toc: true
---

As systems grow from a single process into distributed microservices, observability becomes as important as functionality.

Questions such as:

* Why is API latency increasing?
* Which node is overloaded?
* Is database throughput dropping?
* How many backup jobs are failing?
* Is memory usage growing continuously?

cannot be answered through logs alone.

Modern cloud-native systems rely on three pillars of observability:

1. Metrics
2. Logs
3. Traces

**Prometheus** has become the de-facto standard for metrics collection, while **Grafana** is the most widely used visualization platform.

Together they provide:

* Real-time monitoring
* Historical analysis
* Capacity planning
* Alerting
* Root-cause investigation

In this article, we will:

* Understand Prometheus architecture
* Learn Prometheus metric types
* Instrument a Go application
* Configure Prometheus scraping
* Build Grafana dashboards
* Discuss production best practices

## Why Traditional Monitoring Falls Short

Older monitoring solutions typically relied on:

* Agent-based polling
* Centralized databases
* Proprietary protocols

Challenges included:

* High licensing cost
* Limited scalability
* Vendor lock-in
* Difficult cloud deployment

Cloud-native environments require:

* Dynamic service discovery
* Container awareness
* Horizontal scalability
* Multi-dimensional metrics

Prometheus was designed specifically for these challenges.

## Prometheus Architecture

A Prometheus-based monitoring stack typically looks like:

```text
+-------------+
| Go Service  |
| /metrics    |
+------+------+ 
       |
       | HTTP Scrape
       v
+-------------+
| Prometheus  |
+------+------+ 
       |
       +-----> Alertmanager
       |
       +-----> Grafana
```

Workflow:

1. Application exposes metrics endpoint.
2. Prometheus periodically scrapes endpoint.
3. Metrics stored in time-series database.
4. Grafana queries Prometheus.
5. Alertmanager sends alerts.

## Understanding Time-Series Data

Prometheus stores data as:

```text
Metric Name + Labels + Timestamp + Value
```

Example:

```text
http_requests_total{
    method="GET",
    status="200"
}
```

Stored as:

```text
Metric:
    http_requests_total

Labels:
    method=GET
    status=200

Value:
    12345

Timestamp:
    2026-06-22T10:00:00
```

Each unique label combination creates a separate time series.

## Prometheus Metric Types

Prometheus provides four primary metric types.

### 1. Counter

Counters only increase.

Used for:

* Requests processed
* Jobs completed
* Errors observed

Example:

```go
requestsTotal.Inc()
```

Metric:

```text
http_requests_total 1050
```

Useful PromQL:

```promql
rate(http_requests_total[5m])
```

Calculates requests per second.

### 2. Gauge

Gauge values can increase or decrease.

Examples:

* Memory usage
* Active sessions
* Queue depth

```go
activeUsers.Set(42)
```

Metric:

```text
active_users 42
```

### 3. Histogram

Histograms measure distributions.

Example:

```text
Request latency
```

Buckets:

```text
0.1 sec
0.5 sec
1 sec
2 sec
5 sec
```

Prometheus stores:

```text
request_duration_seconds_bucket
request_duration_seconds_sum
request_duration_seconds_count
```

Useful query:

```promql
histogram_quantile(
  0.95,
  rate(request_duration_seconds_bucket[5m])
)
```

Calculates P95 latency.

### 4. Summary

Summary also measures distributions.

Example:

```text
P50
P90
P99
```

However:

* Not aggregatable across instances
* Less suitable for distributed systems

Modern systems typically prefer Histograms.

## Labels: The Superpower of Prometheus

Without labels:

```text
http_requests_total 1000
```

With labels:

```text
http_requests_total{
    method="GET",
    status="200",
    endpoint="/login"
}
```

Now Prometheus can answer:

```promql
sum by(status)(
    http_requests_total
)
```

or

```promql
sum by(endpoint)(
    rate(http_requests_total[5m])
)
```

Labels transform metrics into powerful analytical tools.

## Prometheus Metric Naming Best Practices

Good:

```text
http_requests_total
```

```text
backup_jobs_failed_total
```

```text
disk_usage_bytes
```

Bad:

```text
reqs
```

```text
memory
```

Prometheus naming guidelines:

```text
<domain>_<metric>_<unit>
```

Examples:

```text
request_duration_seconds
```

```text
storage_used_bytes
```

```text
backup_size_bytes
```

## Adding Prometheus to a Go Project

### Step 1: Install Libraries

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

### Step 2: Create Metrics

**Counter:**

```go
var requestsTotal = prometheus.NewCounter(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Total HTTP requests",
    },
)
```

**Gauge:**

```go
var activeConnections = prometheus.NewGauge(
    prometheus.GaugeOpts{
        Name: "active_connections",
        Help: "Current active connections",
    },
)
```

**Histogram:**

```go
var requestLatency = prometheus.NewHistogram(
    prometheus.HistogramOpts{
        Name:    "request_duration_seconds",
        Help:    "Request latency",
        Buckets: prometheus.DefBuckets,
    },
)
```

### Step 3: Register Metrics

```go
func init() {
    prometheus.MustRegister(
        requestsTotal,
        activeConnections,
        requestLatency,
    )
}
```

### Step 4: Expose Metrics Endpoint

```go
func main() {

    http.Handle(
        "/metrics",
        promhttp.Handler(),
    )

    log.Fatal(
        http.ListenAndServe(":8080", nil),
    )
}
```

### Step 5: Update Metrics

Example middleware:

```go
func handler(
    w http.ResponseWriter,
    r *http.Request,
) {
    start := time.Now()

    requestsTotal.Inc()

    defer func() {
        requestLatency.Observe(
            time.Since(start).Seconds(),
        )
    }()

    w.Write([]byte("hello"))
}
```

Metrics become available at:

```text
http://localhost:8080/metrics
```

## Using Labels in Go

Instead of creating separate metrics:

```text
GET requests
POST requests
DELETE requests
```

Use labels.

```go
var requestCounter = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Request count",
    },
    []string{
        "method",
        "status",
    },
)
```

Update:

```go
requestCounter.
WithLabelValues(
    "GET",
    "200",
).Inc()
```

Produces:

```text
http_requests_total{
    method="GET",
    status="200"
}
```

## Configuring Prometheus

Create `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "go-service"

    static_configs:
      - targets:
        - localhost:8080
```

Start Prometheus:

```bash
./prometheus \
  --config.file=prometheus.yml
```

Visit:

```text
http://localhost:9090
```

## Useful PromQL Queries

### Request Rate

```promql
rate(http_requests_total[5m])
```

### Error Rate

```promql
rate(
  http_requests_total{
    status=~"5.."
  }[5m]
)
```

### Memory Usage

```promql
process_resident_memory_bytes
```

### CPU Usage

```promql
rate(
  process_cpu_seconds_total[5m]
)
```

### P95 Latency

```promql
histogram_quantile(
  0.95,
  sum(
    rate(
      request_duration_seconds_bucket[5m]
    )
  ) by (le)
)
```

## Setting Up Grafana

Run Grafana:

```bash
docker run -d \
  -p 3000:3000 \
  grafana/grafana
```

Access:

```text
http://localhost:3000
```

Default credentials:

```text
admin / admin
```

## Add Prometheus Data Source

Navigate:

```text
Connections
  -> Data Sources
      -> Prometheus
```

URL:

```text
http://localhost:9090
```

Click:

```text
Save & Test
```

## Creating Dashboards

Example panels:

### Traffic

```promql
rate(http_requests_total[5m])
```

### Latency

```promql
histogram_quantile(
  0.95,
  rate(
    request_duration_seconds_bucket[5m]
  )
)
```

### Memory

```promql
process_resident_memory_bytes
```

### Goroutines

```promql
go_goroutines
```

## Go Runtime Metrics

Prometheus automatically exports Go runtime statistics.

Examples:

```text
go_goroutines
go_memstats_heap_alloc_bytes
go_threads
go_gc_duration_seconds
```

Useful Grafana panels:

* Heap usage
* GC pauses
* Goroutine count
* Thread count

These are extremely useful when debugging production issues.

## Monitoring a Distributed Storage System

For systems such as SeaweedFS, backup appliances, or NAS gateways, useful metrics include:

### Metadata Operations

```text
metadata_requests_total
```

### Object Upload Latency

```text
s3_upload_duration_seconds
```

### NFS Requests

```text
nfs_operations_total
```

### Volume Server Health

```text
volume_server_available
```

### Replication Lag

```text
replication_lag_seconds
```

### Cache Hit Ratio

```text
cache_hits_total
cache_misses_total
```

## Alerting Examples

### High Error Rate

```promql
rate(
  http_requests_total{
      status=~"5.."
  }[5m]
) > 10
```

### High Memory

```promql
process_resident_memory_bytes
> 8e9
```

### High Latency

```promql
histogram_quantile(
  0.95,
  rate(
    request_duration_seconds_bucket[5m]
  )
) > 1
```

## Common Mistakes

### High Cardinality Labels

Bad:

```text
user_id
session_id
request_id
```

This can create millions of time series. Avoid labels that are unique per request or per user.

```text
http_requests_total{
    user_id="123456"
}
```

### Missing Histograms

Many teams track only averages. Averages hide tail latency. Always track P50, P90, P95, and P99 using Histograms.

### Over-Instrumentation

Not every variable should become a metric. Monitor:

* Business KPIs
* System health
* Resource consumption
* Error rates

Ignore temporary debug information.

## Conclusion

Prometheus and Grafana have become the standard monitoring stack for cloud-native systems because they are simple, scalable, and highly extensible.

For Go developers, adding monitoring requires only a few lines of code, yet provides deep visibility into:

* Request traffic
* Latency
* Resource usage
* Error rates
* Application health

When building systems such as backup platforms, distributed file systems, object stores, or storage gateways, thoughtful metric design is just as important as application design. Well-designed metrics allow engineers to detect problems early, understand system behavior, and operate large-scale infrastructure with confidence.

The best practice is to instrument metrics from the very beginning of a project rather than treating monitoring as an afterthought.