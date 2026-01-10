# SLIs, SLOs, and SLAs

!!! quote "Source Attribution"
    The core concepts and examples in this document are extracted from **Dark Mode Club's video**. It is highly recommended to watch the full explanation:

    :material-youtube: [Watch: Dark Mode Club - SLIs, SLOs, SLAs](https://www.youtube.com/watch?v=WApyxU4Kaqg)

## 1. Planning

Service Level Objectives (SLOs) distinguish themselves from standard `alerting` and `monitoring` by focusing strictly on **operations that matter to the user**.

### The Planning Workflow

| Step | Description | Example |
| :--- | :--- | :--- |
| **1. Operation** | A customer-facing operation important enough to warrant an SLO. | A new order is generated, submitted, and acknowledged. |
| **2. SLI** (Indicator) | The measurable data needed from the system to compute the SLOs. | Latency in seconds from "order submitted" to "order acknowledged" (derived from logs/telemetry). |
| **3. Aggregation Window** | A timeframe meaningful to the business to spot trends/hotspots. | 8 hours. |
| **4. Target Reliability** | The percentage of success required. (Prod usually > 99%). | 90% (for this example). |
| **5. SLO** (Objective) | The combination of the Indicator and the Target. | New orders are acknowledged within **1 second** and achieve this level of service **90%** of the time. |

### Categories of SLIs

*Based on "Database Reliability Engineering" by Laine Campbell & Charity Majors.*

=== "Latency"

    **Measurement of delay in a communication flow.**

    Usually measured in units of time. Best practice is to measure boundaries:

    * **Upper bound:** "Created within 2 seconds."
    * **Range:** "Created between 1 and 2 seconds."

=== "Availability"

    **Is the service functional?**

    A simple measurement of whether or not a service is generally available given a valid request.

    * *Example:* HTTP 200 OK vs HTTP 500 Error.

=== "Consistency"

    **Quality of Data.**

    "Do users receive the data they expected?"

    !!! warning "The Caching Trap"
        Techniques like caching improve *Latency*, but overdoing it might negatively impact *Consistency* (data freshness).

    **Hybrid Example (Consistency + Latency):**

    > "Users see an online check deposit reflected in their transaction history within 2 seconds."

=== "Throughput"

    **Quantity of data sent/received within a timeframe.**

    * *Example:* "Transactions processed per second: `3`".

    *Context matters:* An overnight batch process measures throughput very differently from a real-time streaming service.

---

## 2. Implementing

**Implementation** is the delivery phase where SLOs impact development efforts.

1.  **Basics:** Ensure SLIs identified in planning are present in logs or time-series DBs.
2.  **Architecture Impact:**
    * Targeting **99%**? Likely no major architectural changes.
    * Targeting **99.999%**? Profound impact on infra and design.

!!! tip "Start Small"
    Don't refactor your entire architecture on day one.

    **Goal 1:** Get actual reliability under measurement.

    **Goal 2:** Iteratively add SLIs to code and hook them into tools like Prometheus.

### Code Instrumentation Example

**Scenario:** User clicks "checkout".

**SLI:** Seconds to process a valid order (derived from logs).

We need 3 distinct log events:

1.  Request initiation.
2.  Success with latency calculation.
3.  Failure events.

```scala title="OrderService.scala"
@throws(classOf[ValidationException])
def checkout(cartItems: ShoppingCart,
             creditCardFrom: CreditCardFrom,
             shippingAdressFrom: AddressFrom): Order = {

  logger.info("checkout requested") // (1)

  try {
    startTime = timer.start() // (2)
    validCard = validateCreditCard(creditCardForm)
    validAddress = validateShippingAdress(shippingAddressFrom)
    order = processOrder(validCard, validAddress, cartItems))
    endTime = timer.end()
    elapsedTime = endTime - startTime

    // Log success with specific latency
    logger.info(s"order ${order.id} successfully took ${elapsedTime}ms") // (3)
    return order
  }
  catch (e: ValidationException) {
    // Log known validation errors (client side errors)
    logger.info("checkout request failed due to a validation error") // (4)
    throw e
  }
  catch (e: ServiceException) {
    // Log server side errors
    logger.error("checkout request failed") // (5)
    throw e
  }
}
```

1.  **Total Requests Marker**: Essential for the denominator in our formula.
2.  **Timer**: Capture high-precision timestamps.
3.  **Success Marker**: Capture the successful ID and the `elapsedTime`.
4.  **Client Error**: These often shouldn't count against your reliability score (depending on policy).
5.  **Server Error**: These definitely count against reliability.

---

## 3. Operations

Once SLIs are implemented, we need visualization (Dashboards) and Reporting.

### The Tooling Stack

* :material-toolbox: **[Sloth](https://sloth.dev/):** Generates Prometheus rules from simple SLO spec files. Saves time vs. hand-coding rules.
* :material-toolbox: **Prometheus:** Standard for metrics-based SLIs.
* :material-toolbox: **Grafana:** Visualizing the data (Supports Prometheus & Loki).
* :material-toolbox: **Loki:** Like Prometheus, but for **logs**. Great for legacy apps where you can't change code easily but have logs.
* :material-toolbox: **Thanos:** Solves Prometheus retention limits (Unlimited history).
* :material-toolbox: **NOBL9:** Turn-key platform for SLOs (if you want to buy vs build).

### Workflow: Logging to Dashboard (Loki path)

1.  **Promtail**: Configure to scrape specific log files containing SLI data.
2.  **Extraction**: Define parser configs (Regex/JSON) in Promtail to isolate fields (latency, status).
3.  **Loki**: Query language extracts the metrics.
4.  **Grafana**: Visualize the query over the **Aggregation Window**.

---

## 4. The Maths

How to calculate the final percentage.

### Scenario A: Standard Calculation

*Objective:* New orders acknowledged within 1s.

*Total Requests:* 110 | *Validation Errors:* 10 | *Service Errors:* 10 | *Slow Requests:* 2

$$
SLO = \frac{Successful Requests}{Total Requests}
$$

**Breakdown:**

* **Valid Requests** = (Total - Validation Errors) = $110 - 10 = 100$
* **Failures** = (Service Errors + Slow Requests) = $10 + 2 = 12$
* **Successes** = (Valid Requests - Failures) = $100 - 12 = 88$

$$
SLO = \frac{88}{100} = 88\%
$$

!!! failure "Result"
    **88% < 90% Target.** We missed the SLO.

### Scenario B: Filtering "Service Errors"

Sometimes, we only want to measure **Latency** performance, excluding crashes (which might be covered by an Availability SLO).

*Formula adjustment:* Remove Service Errors from the denominator.

$$
SLO = \frac{88}{(100 - 10)} = \frac{88}{90} = 97\%
$$

!!! success "Result"
    **97% > 90% Target.** We passed the Latency SLO.

---

## 5. Understanding & Error Budgets

This phase involves periodically reviewing reliability (monthly/quarterly). Data-driven decisions beat gut feelings.

### The Burn Rate

If you target **90%** but achieve **99.9%**, you have a surplus of reliability. This is your **Error Budget**.

* **Under Budget (Failing):** Freeze features, focus on reliability.
* **Over Budget (Success):** Spend the budget on risky changes, refactoring, or speed.

??? abstract "Advanced: Calculating Error Budget"
    Example for a 99.9% Availability Target:

    ```text
    Error Budget = 1 - Availability SLO
    Error Budget = 1 - 99.9% = 0.1%

    Time Calculation (Monthly):
    0.1% of 30 days = 43.2 minutes
    ```

    *Note: Don't worry about this when starting out. Get the measurements working first.*

---

## Resources

**Books**

* :material-library: *Implementing Service Level Objectives* - Alex Hidalgo
* :material-library: *Database Reliability Engineering* - Laine Campbell & Charity Majors
* :material-library: [Google's SRE Book List](https://sre.google/books/)

**Tools**

* :material-toolbox: [Sloth](https://sloth.dev/) (Prometheus SLO Generator)
* :material-toolbox: [Prometheus](https://prometheus.io/)
* :material-toolbox: [Grafana](https://grafana.com/) & [Loki](https://grafana.com/oss/loki/)
* :material-toolbox: [Thanos](https://thanos.io/) (HA Prometheus)
* :material-toolbox: [Nobl9](https://www.nobl9.com/)

**Articles & Lectures**

* [Setting SLOs with Custom Metrics (Google Cloud)](https://cloud.google.com/blog/products/management-tools/setting-slos-with-custom-metrics-in-stackdriver)
* [Google SRE Lecture Series Summary](https://asa55.github.io/class-sre-implementing-slos/)
