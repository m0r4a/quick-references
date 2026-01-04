# PromQL Cheat Sheet

Prometheus Query Language reference for monitoring and alerting.

## Metric Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Counter** | Monotonically increasing value | Request counts, errors |
| **Gauge** | Value that can go up or down | CPU usage, memory, temperature |
| **Histogram** | Distribution of observations | Request latency, response sizes |
| **Summary** | Similar to histogram with quantiles | Precomputed percentiles |

---

## Data Types

**Instant Vector** - Set of time series at a single timestamp
```promql
http_requests_total
```

**Range Vector** - Set of time series over a time range
```promql
http_requests_total[5m]
```

**Scalar** - Simple numeric value
```promql
5
```

**String** - String literal (rarely used)
```promql
"error"
```

---

## Selectors

### Label Matching

=== "Equality"
    ```promql
    # Exact match
    http_requests_total{method="GET"}

    # Multiple labels
    http_requests_total{method="GET", status="200"}
    ```

=== "Inequality"
    ```promql
    # Not equal
    http_requests_total{status!="200"}
    ```

=== "Regex"
    ```promql
    # Regex match
    http_requests_total{status=~"5.."}

    # Negative regex
    http_requests_total{status!~"2.."}
    ```

### Time Ranges

```promql
# 5 minutes
http_requests_total[5m]

# 1 hour
http_requests_total[1h]

# 1 day
http_requests_total[1d]

# Units: s, m, h, d, w, y
```

---

## Rate Functions

!!! warning "Counter vs Gauge"
    Use `rate()` and `increase()` only with counters, never with gauges.

### rate()

Calculate per-second rate over time range:

```promql
# Requests per second
rate(http_requests_total[5m])

# With labels
rate(http_requests_total{job="api"}[5m])
```

### irate()

Instant rate using last two data points (more sensitive to spikes):

```promql
irate(http_requests_total[5m])
```

!!! tip "When to Use"
    - `rate()` - Smooth graphs, alerts, aggregations
    - `irate()` - Volatile metrics, spike detection

### increase()

Total increase over time range:

```promql
# Total requests in last 5 minutes
increase(http_requests_total[5m])

# Requests in last hour
increase(http_requests_total[1h])
```

---

## Aggregation Operators

### Basic Aggregation

```promql
# Sum across all instances
sum(rate(http_requests_total[5m]))

# Average
avg(rate(http_requests_total[5m]))

# Minimum
min(http_requests_total)

# Maximum
max(http_requests_total)

# Count
count(up == 1)
```

### Aggregation by Labels

=== "sum by"
    ```promql
    # Sum per service
    sum by (service) (rate(http_requests_total[5m]))

    # Sum per service and method
    sum by (service, method) (rate(http_requests_total[5m]))
    ```

=== "sum without"
    ```promql
    # Sum, removing instance label
    sum without (instance) (rate(http_requests_total[5m]))
    ```

### All Aggregation Functions

```promql
sum()       # Sum of all values
avg()       # Average
min()       # Minimum
max()       # Maximum
count()     # Count of elements
stddev()    # Standard deviation
stdvar()    # Standard variance
topk(N)     # Top N elements
bottomk(N)  # Bottom N elements
quantile()  # Calculate quantile
```

---

## Binary Operators

### Arithmetic

```promql
# Addition
node_memory_MemTotal_bytes - node_memory_MemFree_bytes

# Percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Division
rate(http_requests_total[5m]) / rate(http_requests_total[5m] offset 1h)
```

### Comparison

```promql
# Greater than
up > 0

# Less than
node_load1 < 0.8

# Equal
http_response_status == 200

# Not equal
http_response_status != 200
```

### Logical

```promql
# AND
up == 1 and on(instance) node_load1 > 0.8

# OR
up == 0 or node_load1 > 1.0

# UNLESS
up == 1 unless node_load1 > 0.8
```

---

## Vector Matching

### One-to-One

```promql
# Match by instance and job
method:http_requests:rate5m{method="GET"} 
  / ignoring(method) 
  method:http_requests:rate5m
```

### Many-to-One

```promql
# Match multiple pods to one node
rate(container_cpu_usage[5m]) 
  * on(instance) group_left(node_name) 
  node_metadata
```

---

## Common Queries

### CPU Usage

```promql
# CPU usage by instance
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# CPU usage by mode
sum by (mode) (rate(node_cpu_seconds_total[5m]))
```

### Memory Usage

```promql
# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Available memory in GB
node_memory_MemAvailable_bytes / 1024^3
```

### Disk Usage

```promql
# Disk usage percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Disk I/O rate
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])
```

### Network

```promql
# Network receive rate in MB/s
rate(node_network_receive_bytes_total[5m]) / 1024^2

# Network transmit rate
rate(node_network_transmit_bytes_total[5m]) / 1024^2
```

### Request Metrics

```promql
# Request rate
sum(rate(http_requests_total[5m]))

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m]))

# Error percentage
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100

# Requests per second by service
sum by (service) (rate(http_requests_total[5m]))
```

---

## Histogram Functions

### histogram_quantile()

Calculate quantiles from histogram buckets:

```promql
# P95 latency
histogram_quantile(0.95,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# P50 latency
histogram_quantile(0.50,
  rate(http_request_duration_seconds_bucket[5m])
)

# P99 latency by service
histogram_quantile(0.99,
  sum by (service, le) (rate(http_request_duration_seconds_bucket[5m]))
)
```

### Histogram Bucket Functions

```promql
# Average request duration
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])
```

---

## Time Functions

### offset

Query data from the past:

```promql
# Current vs 1 hour ago
http_requests_total offset 1h

# Compare current rate to rate 1 day ago
rate(http_requests_total[5m])
/
rate(http_requests_total[5m] offset 1d)
```

### Time Manipulation

```promql
time()              # Current timestamp
minute()            # Current minute (0-59)
hour()              # Current hour (0-23)
day_of_week()       # Day of week (0-6, Sunday=0)
day_of_month()      # Day of month (1-31)
days_in_month()     # Days in current month
month()             # Current month (1-12)
year()              # Current year
```

---

## Useful Functions

### clamp_max / clamp_min

Limit values to range:

```promql
# Cap at 100
clamp_max(cpu_usage, 100)

# Floor at 0
clamp_min(cpu_usage, 0)
```

### round

Round values:

```promql
# Round to nearest integer
round(cpu_usage)

# Round to 1 decimal
round(cpu_usage, 0.1)
```

### absent

Check if metric is missing:

```promql
# Alert if metric disappears
absent(up{job="api"})
```

### label_replace

Modify labels:

```promql
label_replace(
  up,
  "new_label",
  "$1",
  "instance",
  "(.*):.*"
)
```

### predict_linear

Predict future values:

```promql
# Predict disk usage in 4 hours
predict_linear(node_filesystem_free_bytes[1h], 4*3600)
```

---

## Subqueries

Execute query over a time range:

```promql
# Max rate over last hour, calculated per minute
max_over_time(
  rate(http_requests_total[5m])[1h:1m]
)

# Average of P95 over last day
avg_over_time(
  histogram_quantile(0.95, rate(http_request_duration_bucket[5m]))[1d:5m]
)
```

---

## Alert Query Patterns

### Service Down

```promql
# Alert if service is down
up{job="api"} == 0

# Alert if service down for 5 minutes
up{job="api"} == 0 for 5m
```

### High Error Rate

```promql
# Error rate above 5%
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) > 0.05
```

### High Latency

```promql
# P95 latency above 500ms
histogram_quantile(0.95,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
) > 0.5
```

### Resource Exhaustion

```promql
# Disk full in less than 4 hours
predict_linear(node_filesystem_free_bytes[1h], 4*3600) < 0

# Memory usage above 90%
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.9
```

---

## Performance Tips

!!! tip "Query Optimization"
    1. Limit time ranges - use `[5m]` instead of `[1h]` when possible
    2. Filter early - add label selectors to reduce cardinality
    3. Aggregate before calculation - use `sum by` before `rate()`
    4. Avoid regex when possible - use `=` instead of `=~`
    5. Use recording rules for expensive queries

### Good vs Bad

=== "Bad"
    ```promql
    # High cardinality, no filtering
    rate(http_requests_total[1h])
    
    # Unnecessary regex
    http_requests{status=~"200"}
    ```

=== "Good"
    ```promql
    # Filtered and aggregated
    sum by (service) (rate(http_requests_total{job="api"}[5m]))
    
    # Direct match
    http_requests{status="200"}
    ```

---

## Common Pitfalls

!!! danger "Avoid These Mistakes"

    **Using rate() on gauges**
    ```promql
    # WRONG - rate() only for counters
    rate(node_memory_MemAvailable_bytes[5m])
    ```

    **Forgetting range vectors**
    ```promql
    # WRONG - rate() needs range vector
    rate(http_requests_total)

    # CORRECT
    rate(http_requests_total[5m])
    ```

    **Division by zero**
    ```promql
    # Can fail if denominator is zero
    rate(http_errors[5m]) / rate(http_requests[5m])

    # Better - handle zero
    rate(http_errors[5m]) / (rate(http_requests[5m]) + 1)
    ```

---

## Quick Reference

| Pattern | Query |
|---------|-------|
| Request rate | `sum(rate(http_requests_total[5m]))` |
| Error rate % | `sum(rate(http_requests{status=~"5.."}[5m])) / sum(rate(http_requests[5m])) * 100` |
| CPU usage % | `100 - (avg(rate(node_cpu{mode="idle"}[5m])) * 100)` |
| Memory usage % | `(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100` |
| P95 latency | `histogram_quantile(0.95, rate(http_duration_bucket[5m]))` |
| Disk usage % | `(1 - (node_filesystem_avail / node_filesystem_size)) * 100` |
| Top 5 services by requests | `topk(5, sum by (service) (rate(http_requests[5m])))` |
