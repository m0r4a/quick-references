# LogQL Cheat Sheet

Loki Query Language reference for log aggregation and analysis.

## Query Types

| Type | Returns | Example |
|------|---------|---------|
| **Log queries** | Log lines | `{job="nginx"}` |
| **Metric queries** | Numeric values | `rate({job="nginx"}[5m])` |

!!! note "Key Difference from PromQL"
    LogQL queries start with log stream selectors, then optionally apply filters and metrics.

---

## Log Stream Selectors

### Basic Selection

```promql
# Single label
{job="nginx"}

# Multiple labels (AND)
{job="nginx", env="production"}

# Multiple values (OR - use regex)
{job=~"nginx|apache"}
```

### Label Matching Operators

=== "Equality"
    ```promql
    # Exact match
    {job="nginx"}
    
    # Multiple exact matches
    {job="nginx", namespace="default"}
    ```

=== "Inequality"
    ```promql
    # Not equal
    {job!="nginx"}
    
    # Exclude namespace
    {namespace!="kube-system"}
    ```

=== "Regex"
    ```promql
    # Match pattern
    {job=~"nginx.*"}
    
    # Match multiple
    {job=~"nginx|apache"}
    
    # Negative regex
    {job!~"test.*"}
    ```

---

## Log Pipeline

Chain operations to filter and parse logs:

```promql
{job="nginx"}
  | json
  | line_format "{{.method}} {{.path}}"
  | label_format level=`{{ .level | ToUpper }}`
```

---

## Line Filters

### Filter Operators

=== "Contains"
    ```promql
    # Contains text
    {job="nginx"} |= "error"
    
    # Contains any of
    {job="nginx"} |= "error" |= "warning"
    ```

=== "Not Contains"
    ```promql
    # Does not contain
    {job="nginx"} != "debug"
    
    # Exclude multiple
    {job="nginx"} != "debug" != "trace"
    ```

=== "Regex Match"
    ```promql
    # Regex match
    {job="nginx"} |~ "error|warning"
    
    # Case insensitive
    {job="nginx"} |~ "(?i)error"
    ```

=== "Regex Not Match"
    ```promql
    # Negative regex
    {job="nginx"} !~ "debug|trace"
    ```

### Combining Filters

```promql
# Multiple conditions (AND)
{job="nginx"}
  |= "error"
  != "timeout"
  |~ "status=[45].."
```

---

## Parser Expressions

### JSON Parser

```promql
# Parse JSON logs
{job="app"} | json

# Access nested fields
{job="app"} | json | status="200"

# Specific fields
{job="app"} | json level, message, status
```

### Logfmt Parser

```promql
# Parse logfmt (key=value format)
{job="app"} | logfmt

# Filter on parsed fields
{job="app"} | logfmt | level="error"
```

### Pattern Parser

```promql
# Extract with patterns
{job="nginx"} | pattern `<ip> - - <_> "<method> <path> <_>" <status> <_>`

# Named captures
{job="nginx"} | pattern `<ip> - - [<timestamp>] "<method> <path> <protocol>" <status>`
```

### Regexp Parser

```promql
# Named capture groups
{job="nginx"} 
  | regexp `(?P<ip>\S+) .* "(?P<method>\w+) (?P<path>\S+).*" (?P<status>\d+)`

# Filter on extracted fields
{job="nginx"} 
  | regexp `status=(?P<status>\d+)`
  | status >= 400
```

---

## Label Filters

Filter after parsing:

```promql
# Filter on extracted labels
{job="app"} | json | level="error"

# Numeric comparison
{job="app"} | json | status >= 400

# Multiple conditions
{job="app"} | json | level="error" | status >= 500
```

### Comparison Operators

```promql
==  # Equal
!=  # Not equal
>   # Greater than
>=  # Greater or equal
<   # Less than
<=  # Less or equal
```

---

## Formatting

### line_format

Restructure log lines:

```promql
# Custom format
{job="nginx"} 
  | json 
  | line_format "{{.method}} {{.path}} - {{.status}}"

# With functions
{job="app"} 
  | json 
  | line_format "{{.timestamp | ToUpper}} {{.level}}"
```

### label_format

Create or modify labels:

```promql
# Create new label
{job="app"} 
  | json 
  | label_format status_code=`{{.status}}`

# Transform label
{job="app"} 
  | json 
  | label_format level=`{{ .level | ToUpper }}`
```

### Available Functions

```promql
ToLower      # Convert to lowercase
ToUpper      # Convert to uppercase
Replace      # Replace text
Trim         # Trim whitespace
TrimLeft     # Trim left
TrimRight    # Trim right
TrimPrefix   # Remove prefix
TrimSuffix   # Remove suffix
```

---

## Metric Queries

### Range Vector Aggregation

```promql
# Count log lines
count_over_time({job="nginx"}[5m])

# Rate of log lines per second
rate({job="nginx"}[5m])

# Bytes processed per second
bytes_rate({job="nginx"}[5m])

# Bytes processed in time range
bytes_over_time({job="nginx"}[5m])
```

### Functions

| Function | Description | Example |
|----------|-------------|---------|
| `rate()` | Per-second rate | `rate({job="nginx"}[5m])` |
| `count_over_time()` | Count of entries | `count_over_time({job="nginx"}[5m])` |
| `bytes_rate()` | Bytes per second | `bytes_rate({job="nginx"}[5m])` |
| `bytes_over_time()` | Total bytes | `bytes_over_time({job="nginx"}[5m])` |
| `sum_over_time()` | Sum of values | `sum_over_time({job="nginx"} | json | unwrap bytes [5m])` |
| `avg_over_time()` | Average | `avg_over_time({job="nginx"} | json | unwrap duration [5m])` |
| `max_over_time()` | Maximum | `max_over_time({job="nginx"} | json | unwrap duration [5m])` |
| `min_over_time()` | Minimum | `min_over_time({job="nginx"} | json | unwrap duration [5m])` |
| `stdvar_over_time()` | Standard variance | `stdvar_over_time({job="nginx"} | json | unwrap duration [5m])` |
| `stddev_over_time()` | Standard deviation | `stddev_over_time({job="nginx"} | json | unwrap duration [5m])` |
| `quantile_over_time()` | Quantile | `quantile_over_time(0.95, {job="nginx"} | json | unwrap duration [5m])` |

---

## Unwrap

Extract numeric values from logs:

```promql
# Sum response times
sum_over_time(
  {job="nginx"} 
    | json 
    | unwrap duration [5m]
)

# Average bytes
avg_over_time(
  {job="nginx"} 
    | json 
    | unwrap bytes [5m]
)

# P95 latency
quantile_over_time(0.95,
  {job="nginx"} 
    | json 
    | unwrap duration [5m]
)
```

### Unit Conversion

```promql
# Convert bytes to MB
sum_over_time(
  {job="nginx"} 
    | json 
    | unwrap bytes [5m]
) / 1024 / 1024

# Convert milliseconds to seconds
avg_over_time(
  {job="app"} 
    | json 
    | unwrap duration_ms [5m]
) / 1000
```

---

## Aggregation Operators

### Basic Aggregation

```promql
# Sum across streams
sum(rate({job="nginx"}[5m]))

# Average
avg(rate({job="nginx"}[5m]))

# Count
count(count_over_time({job="nginx"}[5m]))

# Min/Max
min(rate({job="nginx"}[5m]))
max(rate({job="nginx"}[5m]))
```

### Group By

```promql
# Group by label
sum by (namespace) (rate({job="nginx"}[5m]))

# Group by multiple labels
sum by (namespace, pod) (rate({job="nginx"}[5m]))

# Group without
sum without (instance) (rate({job="nginx"}[5m]))
```

### Top/Bottom K

```promql
# Top 5 namespaces by log volume
topk(5, sum by (namespace) (rate({job="nginx"}[5m])))

# Bottom 3 services
bottomk(3, sum by (service) (rate({job="app"}[5m])))
```

---

## Common Queries

### Error Rate

```promql
# Error logs per second
sum(rate({job="app"} |= "error" [5m]))

# Error rate by service
sum by (service) (rate({job="app"} |= "error" [5m]))

# Error percentage
(
  sum(rate({job="app"} |= "error" [5m]))
  /
  sum(rate({job="app"}[5m]))
) * 100
```

### HTTP Status Analysis

```promql
# 5xx errors per second
sum(rate({job="nginx"} | json | status >= 500 [5m]))

# Requests by status code
sum by (status) (
  count_over_time({job="nginx"} | json [5m])
)

# 4xx error rate
sum(rate({job="nginx"} | json | status >= 400 | status < 500 [5m]))
```

### Latency Metrics

```promql
# Average response time
avg(
  avg_over_time(
    {job="nginx"} | json | unwrap duration [5m]
  )
)

# P95 latency
quantile_over_time(0.95,
  {job="nginx"} | json | unwrap duration [5m]
)

# P99 latency by endpoint
quantile_over_time(0.99,
  {job="api"} | json | unwrap duration [5m]
) by (path)
```

### Log Volume

```promql
# Total logs per second
sum(rate({namespace="production"}[5m]))

# Logs per second by job
sum by (job) (rate({namespace="production"}[5m]))

# Top 10 noisiest pods
topk(10, 
  sum by (pod) (rate({namespace="production"}[5m]))
)
```

### Pattern Detection

```promql
# Find specific error patterns
{job="app"} |~ "(?i)(error|exception|fatal)"

# Database errors
{job="app"} |~ "(?i)(deadlock|timeout|connection.*failed)"

# Auth failures
{job="app"} |~ "(?i)(unauthorized|forbidden|authentication.*failed)"
```

---

## Alert Patterns

### High Error Rate

```promql
# Alert if error rate > 5%
(
  sum(rate({job="app"} |= "error" [5m]))
  /
  sum(rate({job="app"}[5m]))
) > 0.05
```

### Specific Error Pattern

```promql
# Alert on critical errors
sum(rate({job="app"} |~ "(?i)critical|fatal" [5m])) > 0
```

### Log Volume Anomaly

```promql
# Alert if logs drop significantly
sum(rate({job="app"}[5m]))
<
sum(rate({job="app"}[5m] offset 1h)) * 0.5
```

### High Latency

```promql
# Alert if P95 > 500ms
quantile_over_time(0.95,
  {job="api"} | json | unwrap duration_ms [5m]
) > 500
```

---

## Advanced Patterns

### Detect Anomalies

```promql
# Compare to historical baseline
sum(rate({job="app"} |= "error" [5m]))
/
sum(rate({job="app"} |= "error" [5m] offset 1d))
> 2
```

### Multi-Service Correlation

```promql
# Errors across multiple services
sum by (service) (
  rate({namespace="production"} |= "error" [5m])
)
```

### Time-Based Analysis

```promql
# Hour-over-hour comparison
sum(rate({job="app"}[5m]))
/
sum(rate({job="app"}[5m] offset 1h))
```

---

## Performance Tips

!!! tip "Query Optimization"
    1. **Filter early** - Use stream selectors first
    2. **Limit time ranges** - Smaller ranges = faster queries
    3. **Use specific labels** - Add job/namespace to reduce streams
    4. **Avoid expensive regex** - Use simple filters when possible
    5. **Aggregate when needed** - Don't return raw logs for metrics

### Good vs Bad

=== "Bad"
    ```promql
    # Too broad, no filtering
    {namespace="production"}
    
    # Complex regex on all logs
    {job="app"} |~ ".*error.*warning.*"
    
    # Long time range with no aggregation
    {job="app"}[24h]
    ```

=== "Good"
    ```promql
    # Specific stream, early filter
    {job="app", namespace="production"} |= "error"
    
    # Simple filter
    {job="app"} |= "error"
    
    # Aggregated over reasonable range
    sum(rate({job="app"} |= "error" [5m]))
    ```

---

## Label Operations

### Drop Labels

```promql
# Remove label from result
{job="app"} | json | label_format pod=""
```

### Rename Labels

```promql
# Rename label
{job="app"} | json | label_format new_name=`{{.old_name}}`
```

### Conditional Labels

```promql
# Add label based on condition
{job="nginx"} 
  | json 
  | label_format error_type=`{{ if eq .status "500" }}server{{ else }}client{{ end }}`
```

---

## Comparison: PromQL vs LogQL

| Feature | PromQL | LogQL |
|---------|--------|-------|
| **Data source** | Metrics | Logs |
| **Query start** | Metric name | Label selector |
| **Filters** | Label matchers | Line filters + label matchers |
| **Parsing** | N/A | json, logfmt, regexp, pattern |
| **Aggregation** | On metrics | On log streams |
| **Functions** | Math/statistics | Log-specific + similar to PromQL |

---

## Common Mistakes

!!! danger "Avoid These"

    **Forgetting to parse**
    ```promql
    # WRONG - status not available
    {job="app"} | status="500"

    # CORRECT - parse first
    {job="app"} | json | status="500"
    ```

    **Using wrong function on logs**
    ```promql
    # WRONG - rate() needs range vector
    rate({job="app"})

    # CORRECT
    rate({job="app"}[5m])
    ```

    **Too broad selectors**
    ```promql
    # WRONG - matches everything
    {}

    # CORRECT - specific selector
    {job="app", namespace="production"}
    ```

---

## Quick Reference

| Task | Query |
|------|-------|
| Get logs | `{job="nginx"}` |
| Filter errors | `{job="app"} |= "error"` |
| Parse JSON | `{job="app"} | json` |
| Error rate | `sum(rate({job="app"} |= "error" [5m]))` |
| Log count | `count_over_time({job="app"}[5m])` |
| P95 latency | `quantile_over_time(0.95, {job="app"} | json | unwrap duration [5m])` |
| Top pods by volume | `topk(10, sum by (pod) (rate({job="app"}[5m])))` |
| Pattern search | `{job="app"} |~ "(?i)error|exception"` |
| HTTP 5xx rate | `sum(rate({job="nginx"} | json | status >= 500 [5m]))` |
