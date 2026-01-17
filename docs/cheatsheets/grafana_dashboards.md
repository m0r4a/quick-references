# Grafana Dashboards: A Complete Guide

!!! quote "Source Reference"
    **Based on:** [Grafana dashboards: A complete guide](https://grafana.com/blog/grafana-dashboards-a-complete-guide-to-all-the-different-types-you-can-build/)

    :material-account: **Original Author:** Alexandre de Verteuil

    :material-update: **Last Updated:** Jan 9, 2023

## Introduction

Dashboards are easy to create but difficult to maintain at scale. Without a clear strategy, organizations often face "dashboard sprawl." The following classification scheme helps organize dashboards based on business processes and engineering needs.

---

## 1. Methodologies: USE & REDS

Foundational dashboards for **SREs**. These should be visually simple, uniform, and composed primarily of time-series panels.

=== ":material-server: USE Method"

    **Hardware / Resource Oriented**

    The [USE Method](http://www.brendangregg.com/usemethod.html) (**U**tilization, **S**aturation, **E**rrors) is designed for physical or virtual resources.

    * **Target:** Analyzing machine health and identifying root causes of infrastructure performance issues.
    * **Key Metrics:** CPU, Memory, Disk I/O, Network bandwidth.

=== ":material-traffic-light: REDS Method"

    **Service / User Oriented**

    The REDS metrics (**R**equests, **E**rrors, **D**uration, **S**aturation) derive from the [Four Golden Signals](https://sre.google/sre-book/monitoring-distributed-systems/).

    * **Target:** Microservices and API endpoints.
    * **Usage:** Primary candidates for alerting; these metrics serve as a proxy for the actual user experience.

---

## 2. Navigation & Flows

### Overview and Drill Down

Implemented using [Dashboard Links](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/manage-links/) or [Data Links](https://grafana.com/docs/grafana/latest/panels-visualizations/configure-data-links/) to create a hierarchy.

* **Overview:** High-level aggregated metrics for the entire infrastructure or fleet.
* **Drill Down:** granular metrics focusing on a single component or instance.

### Business Journey

Visualizations that track high-level business logic rather than technical metrics.

* **Scope:** Customer acquisition funnels, supply chain logistics, or physical operations (e.g., IoT manufacturing lines).

### The Home Dashboards

Custom entry points configured at the Organization, Team, or User level.

* **Strategy:** Can range from simple row additions to "Enterprise" setups with team-specific layouts.
* **Components:** Useful for displaying admin contact info and dynamic [Dashboard Lists](https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/dashboard-list/).

---

## 3. Development Workflows

### Research & Development (R&D)

Sandboxes for iterative dashboard design and "work-in-progress" (WIP).

* **Organization:** Isolate in folders like `SRE R&D` or `AIOps Drafts`.
* **Best Practice:** Avoid using production tags to prevent drafts from polluting global search results.

### Metrics Exploration

Abstract, reusable dashboards for browsing data when specific metric names are unknown.

**Design Pattern:** Use [Variables](https://grafana.com/docs/grafana/latest/dashboards/variables/) to select metric prefixes, with panels repeating automatically.

**PromQL Aggregation Templates:**

```promql
# 1. Average Value
avg without (instance) ($metric)

# 2. Total Sum
sum without (instance) ($metric)

# 3. Average Per-Second Rates
avg without (instance) (rate($metric[$__rate_interval]))

# 4. Total Rate Sum
sum without (instance) (rate($metric[$__rate_interval]))
```

---

## 4. Operational Views

### Alerts Analysis

Visualizes the *history* and state of alerts, typically using the [State Timeline](https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/state-timeline/) panel.

**Visualizing Alert States (State Timeline):**

Map states to integers for visualization (e.g., 3=Meta, 2=Firing, 1=Pending).

```promql
max by (alertname,alertstate) (
  3 * max_over_time(ALERTS{alertname="AlwaysFiring"}[$__interval])
  or
  2 * max_over_time(ALERTS{alertstate="firing"}[$__interval])
  or
  max_over_time(ALERTS{alertstate="pending"}[$__interval])
)
```

**Counting Firing Alerts (Status History):**

```promql
count by (alertname) (max_over_time(ALERTS{alertstate="firing"}[$__interval]))
```

### Issue Dashboards

Ephemeral dashboards created for specific incident investigations.

* **Lifecycle:** Temporary; should be archived or deleted after resolution.
* **Naming Convention:** Include timestamp or Incident ID (e.g., `2023-10-Incident-Database-Lock`).

### Meta-Monitoring

Observability for the observability stack itself (:fontawesome-brands-linux: Prometheus, :material-text-search: Loki, :material-bell: Alertmanager).

* **Access:** Restricted to Platform Admins.
* **Purpose:** Ensure the monitoring pipeline is healthy (e.g., scrape failures, rule evaluation times).

---

## 5. Visualization Formats

### Big Screen (Wall TV)

Optimized for open office displays or NOCs.

* **UX Design:** High contrast, large fonts, instant readability (Stat panels, Gauges).
* **Focus:** Identifying *what* is broken immediately, rather than explaining *why*.

### Reports

*Requires [Grafana Enterprise](https://grafana.com/products/enterprise/).*

Layouts designed for **PDF export** and email distribution to stakeholders.

* **Constraints:** Must be tuned for static rendering; interactive elements do not translate well to print/PDF.

---

## 6. Automation & Sourcing

### Prebuilt Dashboards

* **Sources:** Community Plugins, Cloud Integrations, and [Mixins](https://monitoring.mixins.dev/) (Jsonnet bundles).
* **Repository:** [Grafana Dashboards Public Repo](https://grafana.com/grafana/dashboards/).

### Dashboards as Code (IaC)

Managing dashboards via API or Terraform to ensure version control and reproducibility.

**Automation Methods:**

* **File Provisioning:** Placing JSON files in the server's provisioning directory (CI/CD friendly).
* **HTTP API:** Scripted interactions.

!!! info "API Reference"
    Key endpoints for automation:

    * :material-api: [Auth API](https://grafana.com/docs/grafana/latest/http_api/auth/)
    * :material-api: [Dashboard API](https://grafana.com/docs/grafana/latest/http_api/dashboard/)
    * :material-api: [Folder API](https://grafana.com/docs/grafana/latest/http_api/folder/)

**Tooling:**

* [Grafonnet](https://github.com/grafana/grafonnet-lib) (Jsonnet library)
* [Grizzly](https://github.com/grafana/grizzly) (Dashboards as Code tool)
* [Terraform Provider](https://registry.terraform.io/providers/grafana/grafana/latest/docs)

---

## 7. Organization Strategy

### "The Watchtower" Structure

Recommended folder taxonomy for self-hosted instances to maintain order:

| Folder | Purpose |
| :--- | :--- |
| **Archive** | Deprecated dashboards (kept for query recovery). |
| **Issues** | Investigation dashboards (Prefix: `yyyy-mm-dd`). |
| **User Prod** | Personal dashboards promoted to production use. |
| **User R&D** | Personal drafts and experiments. |
| **Meta-monitoring** | Stack health and Metrics Exploration. |
| **General** | Shared, stable production dashboards. |

---

## References

* :material-file-document: [Best practices for managing dashboards](https://grafana.com/docs/grafana/latest/dashboards/manage-dashboards/)
* :material-folder: [Dashboard folders documentation](https://grafana.com/docs/grafana/latest/dashboards/manage-dashboards/#dashboard-folders)
* :material-lock: [Permissions & Roles](https://grafana.com/docs/grafana/latest/administration/roles-and-permissions/)
