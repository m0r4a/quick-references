# Grafana Dashboards: A Complete Guide

!!! quote "Article Metadata"
    **Title:** Grafana dashboards: A complete guide to all the different types you can build

    :material-account: **Author:** Alexandre de Verteuil

    :material-calendar: **Published:** 2022-06-07 • 16 min

    :material-update: **Updated:** Jan 9, 2023

    *Editor’s note: This blog was updated on Jan. 9, 2023, to reflect our latest releases.*

## Introduction

There is one universal truth about using Grafana: Dashboards are easy to create, but not-so-easy to organize.
As organizations scale, there’s a high risk of unchecked dashboard sprawl, when dashboards become an unmanageable mess.
As the number of users increases, so does their dashboard output.

Our guide to dashboard management gives an overview of features that help with organizing dashboards, but there are still two pain points:

* There are not a lot of details, examples, or opinions on how users can categorize and classify their Grafana dashboards.
* Grafana’s dashboard folder structure, as it is currently implemented, is limited. You can’t create subfolders, only first-level folders.

In this article, I will list and describe all the different types of Grafana dashboards that currently exist.

---

## 1. Methodologies: USE & REDS

These Grafana dashboards are built on the USE and REDS methods.
The USE and REDS dashboards are particularly useful for site reliability engineers.
They are visually very simple and uniform, mostly made up of time series panels.

=== "USE Method"

    **Hardware Oriented**

    The USE metrics (**U**tilization, **S**aturation, **E**rrors) are oriented towards hardware resources of your infrastructure.

    * They help you understand how your machines are doing and what the cause of a problem might be.

=== "REDS Method"

    **Service Oriented**

    The REDS metrics (**R**equests, **E**rrors, **D**uration, **S**aturation — also known as the Four Golden Signals) are service-oriented, and they are also likely the ones you will want to alert on.

    * REDS dashboards tell you how your services are performing and are a good proxy for your users’ experience.

!!! tip "Standardization"
    By standardizing dashboards across an organization using those methodologies, operators can interpret dashboards across different teams efficiently.
    Designating Grafana dashboards as “USE” and “REDS” in their title can also help find the right dashboard in a given context.

---

## 2. Navigation & Flows

### Overview and Drill Down

In Grafana, drill downs from aggregated views to detailed views are implemented by linking between different Grafana dashboards.

* **Overview Dashboard:** Displays aggregated metrics for an entire infrastructure or service.
* **Detailed Dashboard:** Shows more detailed metrics about a subset of an infrastructure or a single component instance.

*Implementation:* This is usually implemented using dashboard links, data links, and URL variables.

### Business Journey/Process Flow

Thousands of businesses use Grafana dashboards to visualize their customer acquisition flows, supply chains, and operations.

*Examples:*

* Manufacturing plant efficiency.
* Visualizing CAN IoT data to monitor vehicles and machinery.
* Supporting sustainability.

### The Home Dashboards

You can easily customize the home dashboard in Grafana to provide orientation to your users.
The home dashboard can be set at the Organization level, the Team level, or the User level in Grafana.

There are three approaches to home dashboards:

1.  **The Light Approach:** Keep the original content, but add a row at the top with your own dashboard lists.
2.  **The Heavy Approach:** Build an entirely custom home dashboard.
3.  **The Enterprise Approach:** Build a custom home dashboard for each team.

**What to put on a home dashboard:**

* Information in a text panel explaining who is managing this Grafana instance, what is monitored here, who to contact for help.
* Dashboard list panels managed dynamically using tags.

---

## 3. Development Workflows

### Research & Development (R&D)

Dashboard development is an iterative process. Users should have a place to save their test and work-in-progress dashboards.

* **Organization:** Folders with the user’s or team’s name can help organize those unfinished Grafana dashboards — for example “AIOps drafts” or “SRE R&D” or “Cloud Platform WIP”.

* **Tagging:** R&D dashboards should not have tags in common with production dashboards to avoid them appearing in dashboard lists and links.

### Metrics Exploration

When I’m not familiar with the metrics available for a system, I sometimes build a metrics exploration dashboard.
Made of templated generic queries and repeated panels, these Grafana dashboards allow me to browse and discover useful metrics in a given data source.

* **Purpose:** It answers questions like "I added a metrics scrape job for a system. What metrics are available?".

* **Design:** This dashboard is as abstract and generic as possible. Variables provide a way to categorize and list metrics based on their prefix.

**Example Layout:**
Four panels repeat over each metric, aggregating values in four different ways:

```promql
# Average
avg without (instance) ($metric)

# Sum
sum without (instance) ($metric)

# Average rates
avg without (instance) (rate($metric[$__rate_interval]))

# Rate totals
sum without (instance) (rate($metric[$__rate_interval]))
```

---

## 4. Specialized Visualization Types

### Big Screen Dashboards

These are dashboards designed to be displayed on a big screen in an open workspace.

* **Visuals:** They are often made of stat, gauge, and bar gauge panels. They usually make use of value thresholds with color-coding as well.
* **Intent:** One of the design intents for this kind of dashboard is to provide an instant emotional reading, not depth of detail.
* **Content:** If the dashboard displays status or alert information, it will often specify *what* is broken, but not *why*.

### Reports Dashboards

*Note: Reporting is a Grafana Enterprise-only feature.*

Similar to the big screen dashboards, a reports dashboard provides a quick overview, except the output media is a **PDF** attached to an email.

* **Use Case:** To provide analysis and overviews to high-level executives who don’t typically log into Grafana in their day-to-day job.
* **Design:** Dashboards used for monitoring don’t usually translate naturally to a PDF, so users will typically create dashboards specifically for reports and fine-tune the layout until it looks good on the PDF.

### Demo and Training

These Grafana dashboards are present all over the place here at Grafana Labs.
They help us demonstrate the value of Grafana, and they are sources of inspiration, examples, and best practices.

* **Data Sources:** They sometimes connect to the TestData data source plugin, or a data source with data generators that generate predictable metrics.

---

## 5. Alerting & Troubleshooting

### Alerts Analysis

Here’s a perfect use case for the new state timeline panel released in Grafana 8.
Prometheus generates a synthetic alerts metric which makes the history of alerts queryable.

**Top Panel (State Timeline):**
I crafted this PromQL expression to return 3 for my AlwaysFiring meta-alert, 2 for firing alerts, and 1 for pending alerts.

```promql
max by (alertname,alertstate) (
  3 * max_over_time(ALERTS{alertname="AlwaysFiring"}[$__interval])
  or
  2 * max_over_time(ALERTS{alertstate="firing"}[$__interval])
  or
  max_over_time(ALERTS{alertstate="pending"}[$__interval])
)
```

**Bottom Panel (Status History):**
This uses the "Yellow-Red (by value)" color scheme.

```promql
count by (alertname) (max_over_time(ALERTS{alertstate="firing"}[$__interval]))
```

### Issue Dashboards

This is a type of Grafana dashboard created for investigating a specific issue.
Their use is scoped to a limited time, after which they become obsolete or stale.

* **Why not just Explore?** If it’s a hard-to-diagnose issue you have been chasing for weeks or months, there is a case for setting up a folder for such dashboards.
* **Naming:** You may also add a timestamp or an issue number in the dashboard title.

### Meta-Monitoring

Meta-monitoring dashboards display metrics about your organization’s monitoring and observability stack.
They’re saved in a separate folder because the audience is limited to the observability platform admins.

* **Metrics:** Grafana, Prometheus, the Grafana Agent, Pushgateway, Alertmanager, Grafana Loki, etc.

---

## 6. Sourcing & Automation

### Prebuilt Dashboards & Mixins

There are Grafana dashboards made by other people and shared with the community.

* **Plugins:** Some Grafana data source plugins and Grafana Cloud integrations include prebuilt dashboards (e.g., Grafana Enterprise Metrics).
* **Mixins:** Collections of Grafana dashboards, Prometheus alerts, and recording rules built from the collective experience of a system’s community. Example: Kubernetes mixin.
* **Public Repository:** Shared dashboards will get you maybe 50% or 90% of the way to your desired visualizations, but rarely work 100% right out of the box.

### Dashboards as Code

Dashboards can be generated from code and automatically published to Grafana.

* **Tools:** `grafonnet-lib`, `grizzly`, Terraform provider.
* **Provisioning:** This is a tooling-agnostic facility that allows you to keep dashboards’ source of truth outside of Grafana and to update them by dropping files on the server’s filesystem.

---

## 7. Organization Example: "The Watchtower"

Here is how I organize dashboard folders in my Grafana instance:

| Folder Name | Description |
| :--- | :--- |
| **Alexandre archive** | Graveyard of dashboards I no longer use but want to keep around in case I want to re-use some queries or visualizations. |
| **Alexandre issues** | Dashboards created for investigating specific problems. Titles are prefixed with a “yyyy-mm-dd” formatted date. |
| **Alexandre prod** | Dashboards I use regularly but that are not useful for my brother. |
| **Alexandre R&D** | Dashboard drafts, work-in-progress, tests. |
| **Meta-monitoring** | Grafana, Grafana Loki, Prometheus status, Metrics exploration. |
| **General** | Production dashboards that are useful to both my brother and me. |

---

## Resources

**Tools & Libs**

* :material-github: [grafonnet-lib](https://github.com/grafana/grafonnet-lib)
* :material-github: [grizzly](https://github.com/grafana/grizzly)
* :material-terraform: [terraform-provider-grafana](https://github.com/grafana/terraform-provider-grafana)

**Documentation**

* :material-file-document: [Grafana HTTP API](https://grafana.com/docs/grafana/latest/http_api/)
* :material-file-document: [Best practices for managing dashboards](https://grafana.com/docs/grafana/latest/dashboards/manage-dashboards/)
