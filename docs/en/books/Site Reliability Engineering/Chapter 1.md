# Chapter 1 - Introduction

**"Hope is not a strategy."** — Traditional SRE saying

This document breaks down the fundamental shift from the traditional "Sysadmin" model to Google's **Site Reliability Engineering (SRE)** model. It outlines the core tenets necessary to run large-scale systems efficiently.


---

## The Problem: The Sysadmin Approach

Historically, companies treated "Development" and "Operations" as discrete, isolated teams. This model suffers from specific pitfalls:

* **Linear Scaling Costs:** As traffic grows, you need more humans to manage events and updates.
* **Misaligned Incentives:** Developers want velocity (new features), while Ops want stability (no changes).
* **The "Trench Warfare" Effect:** Ops creates gatekeeping to slow down changes, while Devs try to bypass gates with "flag flips" or incremental updates to sneak features in.

!!! failure "The Cost of Separation"
    The split between Dev and Ops leads to a pathology of communication breakdowns, conflicting goals, and a lack of mutual trust.

---

## The Solution: The SRE Approach

Google's definition is simple: **"SRE is what happens when you ask a software engineer to design an operations team."**

### Team Composition

The SRE team is a synthesis of different skill sets, typically split into two buckets:

1.  **50–60% Google Software Engineers:** Hired via standard SWE procedures.
2.  **40–50% System Experts:** Candidates with 85–99% of SWE skills but with rare technical expertise, specifically UNIX system internals and Layer 1–3 networking.

### The Scaling Advantage

Unlike the sysadmin model, SRE is designed to scale **sublinearly**.

* SREs are bored by manual tasks and have the skillset to automate them.
* The goal is to create systems that are **automatic**, not just automated.
* The team repairs itself; as the service grows, SREs focus on engineering solutions rather than adding more bodies to the rotation.

---

## The 50% Rule

To prevent the team from drowning in operational load (toil), Google enforces a strict cap on "ops" work.

!!! quote "Code or Drown"
    The team tasked with managing a service needs to code or it will drown.

* **The Cap:** SREs spend a maximum of **50%** of their time on ops (tickets, on-call, manual tasks).
* **The Remaining 50%:** Must be spent on actual development and engineering projects to make the service stable and operable.
* **Enforcement:** If ops load exceeds 50%, the burden is shifted back to the Development team, or staff is added specifically to handle the load without assigning them extra ops duties.

---

## DevOps vs. SRE

Think of it in object-oriented terms: **class SRE implements DevOps**.

* **DevOps:** A general philosophy involving heavy automation and cross-functional cooperation.
* **SRE:** A specific implementation of DevOps with idiosyncratic extensions (like error budgets).

---

## The Tenets of SRE

While workflows vary, all SRE teams adhere to these core principles to maintain their engineering focus.

### 1. Durable Focus on Engineering

We monitor the operational load to ensure we have time to code.

* **On-Call Limit:** Maximum of **2 events** per 8–12 hour shift.
* **Why?** More than 2 events prevents thorough investigation and postmortems; fewer than 1 event is a waste of time.
* **Postmortems:** Written for all significant incidents (even those that didn't page) to expose faults and apply engineering fixes.

### 2. Error Budgets (Velocity vs. Stability)

We resolve the conflict between Dev and Ops by accepting that **100% availability is the wrong target**.


$$ Error Budget = 1 - Availability Target $$

* **The Reality:** Users cannot distinguish between 100% and 99.999% availability due to the noise of their own ISP, WiFi, or hardware failures.
* **The Strategy:** The business defines an availability target (e.g., 99.99%). The remaining 0.01% is the **Error Budget**.
* **Spending the Budget:** This budget can be spent on risky feature launches or experiments.
* **Consequence:** If the budget is exhausted, releases stop until the system stabilizes. This aligns Dev and Ops incentives.

### 3. Monitoring

Monitoring shouldn't require humans to stare at screens. There are only three valid outputs:

| Type | Definition | Action Required |
| :--- | :--- | :--- |
| **:material-bell-ring: Alerts** | Something is broken or about to break. | **Immediate** human action required. |
| **:material-ticket-confirmation: Tickets** | Something needs attention, but it can wait a few days. | **Delayed** human action required. |
| **:material-math-log: Logging** | Recorded for forensics/diagnostics. | **No** action unless prompted by an investigation. |

### 4. Emergency Response

Reliability is a function of Mean Time to Failure (MTTF) and Mean Time to Repair (MTTR).

* **Objective:** Lower MTTR.
* **Playbooks:** Recording best practices in a playbook produces roughly **3x improvement in MTTR** compared to "winging it".
* **Practice:** Use exercises like the "Wheel of Misfortune" to prepare engineers.

### 5. Change Management

* **Fact:** Roughly **70%** of outages are due to changes in a live system.
* **Mitigation:**
    * Progressive rollouts.
    * Quick/Accurate problem detection.
    * Safe rollback capabilities.

### 6. Capacity Planning

We must ensure sufficient capacity for organic (natural adoption) and inorganic (launches/marketing) growth.

!!! tip "Mandatory Steps"
    1.  **Forecast:** Accurate organic demand forecasting beyond lead times.
    2.  **Inorganic:** Incorporation of business-driven events (launches).
    3.  **Load Testing:** Regular testing to correlate raw capacity (disk/cpu) to service capacity.

### 7. Provisioning

Provisioning combines change management and capacity planning. It must be fast, correct, and only done when necessary because capacity is expensive.

### 8. Efficiency and Performance

SREs are ultimately responsible for utilization and cost.

* **The Lever:** Provisioning strategy is a massive lever on total service costs.
* **Performance:** A slow system eventually stops serving (infinite slowness). Improving performance adds capacity and efficiency.

**Next Steps:**
Check out the **[Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)** to see how modern monitoring implements the Alerting vs. Logging distinction mentioned in Tenet 3.
