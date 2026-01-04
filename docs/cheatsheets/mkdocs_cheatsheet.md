# Mkdocs - Material cheatsheet

Complete reference for all enabled extensions and features.

## Table of Contents

Use `[TOC]` or it auto-generates from headers with `permalink: true`.

---

## Code Blocks

### Basic Syntax Highlighting

Supported languages: `promql`, `go`, `bash`, `python`, `yaml`, `json`, `dockerfile`, etc.

```promql
# PromQL query
rate(http_requests_total{job="api"}[5m])
```

```go
// Golang example
func main() {
    fmt.Println("Hello World")
}
```

```bash
# Bash commands
kubectl get pods -n monitoring
systemctl status prometheus
```

**Markdown source:**
````markdown
```promql
rate(http_requests_total[5m])
```
````

---

### Code Blocks with Line Numbers

Add line numbers to any code block:

```python linenums="1"
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

result = fibonacci(10)
print(f"Result: {result}")
```

**Markdown source:**
````markdown
```python linenums="1"
def fibonacci(n):
    return n if n <= 1 else fibonacci(n-1) + fibonacci(n-2)
```
````

---

### Highlighting Specific Lines

Highlight important lines:

```yaml hl_lines="2 4-6"
apiVersion: v1
kind: Service  # This line is highlighted
metadata:
  name: prometheus  # These lines
  namespace: monitoring  # are also
  labels:  # highlighted
    app: prometheus
```

**Markdown source:**
````markdown
```yaml hl_lines="2 4-6"
apiVersion: v1
kind: Service
```
````

---

### Code with Title

```python title="fibonacci.py"
def fibonacci(n):
    """Calculate fibonacci number."""
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

**Markdown source:**
````markdown
```python title="fibonacci.py"
def fibonacci(n):
    return n
```
````

---

### Code Annotations

Add inline explanations to code:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx  # (1)!
spec:
  replicas: 3  # (2)!
  selector:
    matchLabels:
      app: nginx
```

1. The name of your deployment
2. Number of pod replicas

**Markdown source:**
````markdown
```yaml
metadata:
  name: nginx  # (1)!
```

1. The name of your deployment
````

---

## Inline Code Highlighting

Use inline syntax highlighting with backticks and language: `#!python range(10)` or `#!bash curl -X GET`.

**Markdown source:**
```markdown
Use `#!python range(10)` for inline code.
```

---

## Admonitions (Info Boxes)

### Basic Admonitions

!!! note
    This is a note admonition. Use it for general information.

!!! tip
    This is a tip admonition. Use it for helpful suggestions.

!!! warning
    This is a warning admonition. Use it for important cautions.

!!! danger
    This is a danger admonition. Use it for critical warnings.

**Markdown source:**
```markdown
!!! note
    This is a note admonition.

!!! warning
    This is a warning admonition.
```

---

### Custom Titles

!!! tip "PromQL Pro Tip"
    Always use `rate()` for counters, never raw values.

**Markdown source:**
```markdown
!!! tip "PromQL Pro Tip"
    Always use `rate()` for counters.
```

---

### Collapsible Admonitions

Use `???` instead of `!!!` to make them collapsible:

??? note "Click to expand"
    This content is hidden by default. Users must click to see it.

??? danger "Critical Performance Issue"
    Querying large time ranges without aggregation can crash Prometheus.
    
    ```promql
    # Bad - queries all series
    http_requests_total
    
    # Good - aggregated
    sum(rate(http_requests_total[5m]))
    ```

**Markdown source:**
```markdown
??? note "Click to expand"
    This content is hidden by default.
```

---

### Inline Blocks (Already Expanded)

???+ tip "Pre-expanded Tip"
    Use `???+` to make collapsible blocks expanded by default.

**Markdown source:**
```markdown
???+ tip "Pre-expanded Tip"
    Content here starts visible.
```

---

## Content Tabs

Compare different approaches side-by-side:

=== "PromQL"
    ```promql
    rate(http_requests_total[5m])
    ```

=== "LogQL"
    ```promql
    rate({job="nginx"}[5m])
    ```

=== "Bash"
    ```bash
    curl -s localhost:9090/api/v1/query?query=up
    ```

**Markdown source:**
```markdown
=== "PromQL"
    ```promql
    rate(http_requests_total[5m])
    ```

=== "LogQL"
    ```promql
    rate({job="nginx"}[5m])
    ```
```

---

### Nested Tabs

=== "Monitoring"
    === "Prometheus"
        ```yaml
        scrape_configs:
          - job_name: 'api'
            static_configs:
              - targets: ['localhost:8080']
        ```
    
    === "Grafana"
        ```json
        {
          "datasource": "Prometheus",
          "expr": "rate(http_requests[5m])"
        }
        ```

=== "Logging"
    === "Loki"
        ```yaml
        auth_enabled: false
        server:
          http_listen_port: 3100
        ```

**Markdown source:**
```markdown
=== "Monitoring"
    === "Prometheus"
        Content here
    === "Grafana"
        Content here
```

---

## Keyboard Keys

Press ++ctrl+c++ to copy or ++ctrl+shift+p++ to open command palette.

Common shortcuts:
- Save: ++ctrl+s++
- Quit: ++ctrl+q++
- Search: ++ctrl+f++
- Terminal: ++ctrl+grave++

**Markdown source:**
```markdown
Press ++ctrl+c++ to copy.
Press ++ctrl+shift+p++ for command palette.
```

---

## Snippets (Reusable Content)

Create reusable content in separate files:

**File: `docs/snippets/common-queries.md`**
```promql
# CPU Usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**Include in any page:**
```markdown
--8<-- "snippets/common-queries.md"
```

**Markdown source:**
```markdown
--8<-- "snippets/filename.md"
```

---

## Combining Features

Here's a complex example using multiple features:

!!! example "Complete Monitoring Setup"
    
    === "Prometheus Config"
        ```yaml title="prometheus.yml" linenums="1" hl_lines="3-4"
        global:
          scrape_interval: 15s  # (1)!
        scrape_configs:
          - job_name: 'kubernetes-pods'  # (2)!
            kubernetes_sd_configs:
              - role: pod
        ```
        
        1. How often to scrape targets
        2. Auto-discover Kubernetes pods
    
    === "Query Examples"
        ```promql
        # Request rate per service
        sum by (service) (rate(http_requests_total[5m]))
        
        # P95 latency
        histogram_quantile(0.95, 
          sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
        )
        ```
    
    !!! tip
        Test queries in Prometheus UI before adding to Grafana dashboards.

---

## Best Practices Summary

???+ success "Quick Reference"
    
    **Code Blocks:**
    - Use line numbers for long snippets: `linenums="1"`
    - Highlight important lines: `hl_lines="2 4-6"`
    - Add titles for context: `title="config.yaml"`
    
    **Organization:**
    - Use tabs for comparing alternatives
    - Use admonitions for warnings/tips
    - Make long sections collapsible with `???`
    
    **Keyboard Shortcuts:**
    - Format: `++ctrl+shift+p++`
    - Separate keys with `+`
    
    **Inline Code:**
    - Add syntax highlighting: `` `#!python print()` ``

---

## Common Admonition Types

!!! note
    General information

!!! abstract
    Summary or TL;DR

!!! info
    Additional context

!!! tip
    Helpful suggestions

!!! success
    Positive outcome

!!! question
    Questions or help needed

!!! warning
    Important cautions

!!! failure
    Errors or failures

!!! danger
    Critical warnings

!!! bug
    Known issues

!!! example
    Code examples

!!! quote
    Citations or quotes

---

## Markdown Source Template

```markdown
# Your Page Title

## Section

Regular paragraph with `inline code`.

### Code Example

```bash
command --flag value
```

### Info Box

!!! tip "Helpful Tip"
    Your tip content here.

### Tabs

=== "Option 1"
    Content for option 1

=== "Option 2"
    Content for option 2

### Keyboard

Press ++ctrl+c++ to copy.
```

---
