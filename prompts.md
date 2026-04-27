# Agentfest AI Innovation Summit — Prompt Templates

Use these prompts with **Genie Code** (or any Databricks agent) to build your data assets step by step.
Before you start, replace `<your_catalog>` and `<your_schema>` with your own Unity Catalog values throughout.

---

## Prerequisites

| Setting | Your Value |
|---|---|
| Catalog | `<your_catalog>` |
| Schema | `<your_schema>` |
| Tables available | `members`, `eligibility`, `wa_hca_eligibility`, `providers`, `prior_authorizations`, `claims`, `appeals` |

---

## Step 1 — Create Metric View

> **What this builds:** A Unity Catalog Metric View that acts as the semantic layer and source of truth for all downstream dashboards and Genie Spaces.

**Prompt:**

```
I have healthcare data in the <your_catalog>.<your_schema> schema with the following tables:
members, eligibility, wa_hca_eligibility, providers, prior_authorizations, claims, appeals.

My business requirement is:
Enrollment teams and compliance officers need to know how many members are active at any point
in time, plan type mix, and how WA Medicaid enrollment trends by county. This is foundational
for per-member-per-month (PMPM) cost analysis.

Create a metric view in <your_catalog>.<your_schema> with the following measures and dimensions:

Measures:
1. Active Member Count — COUNT_IF(is_active = true) on the eligibility table — members with coverage right now
2. New Enrollments — COUNT(eligibility_id) on the eligibility table — coverage starts in a given period
3. Active WA Medicaid Members — COUNT(DISTINCT state_member_id) WHERE eligibility_status = 'Active' on the wa_hca_eligibility table — WA Medicaid headcount for PMPM base

Dimensions:
1. Plan Type — plan_type on the eligibility table — Commercial, Medicaid, Medicare, CHIP
2. WA County — county on the wa_hca_eligibility table — King, Pierce, Clark, Spokane, etc.
3. Enrollment Month — DATE_TRUNC('MONTH', coverage_start) on the eligibility table — month-over-month trend slicing

Also review the tables and suggest any additional measures or dimensions that would be
useful for this business requirement.
```

---

## Step 2 — Create Dashboard

> **What this builds:** An AI/BI dashboard using the metric view from Step 1 as the data source.

**Prompt:**

```
Using the metric view created in Step 1 in <your_catalog>.<your_schema>, create an AI/BI
dashboard for the Member Enrollment & Eligibility use case.

Include the following visualizations:
1. Active Members by Plan Type — show the book of business mix (Commercial, Medicaid, Medicare, CHIP)
2. Monthly New Enrollments — show growth rate over time as a trend
3. WA Medicaid Members by County — show geographic coverage spread across WA counties
4. Net Enrollment Change — enrollments minus lapses per month
5. PMPM Enrollment Base — latest active WA Medicaid member count with a 6-month trend

Also identify any other useful KPIs or visualizations based on the available data that would
be valuable to an enrollment team or compliance officer at a health plan.
```

---

## Step 3 — Create Genie Space

> **What this builds:** A conversational analytics interface over the metric view, pre-loaded with instructions, SQL examples, and benchmarks so business users can ask questions in plain English.

**Prompt:**

```
Using the metric view created in Step 1 in <your_catalog>.<your_schema>, create a Genie Space
for the Member Enrollment & Eligibility use case.

Requirements:
1. Set the metric view as the primary data source for the Genie Space
2. Enable tool calling so Genie can query the metric view directly
3. Follow Genie Space best practices for configuration
4. Populate the following sections with appropriate content:

   General Instructions:
   - The audience is enrollment teams and compliance officers at a health plan
   - Focus on member counts, plan type mix, and WA Medicaid county trends
   - Always clarify whether metrics refer to active members or all members
   - PMPM (per-member-per-month) is the primary unit of analysis

   Joins:
   - Document how eligibility joins to members on member_id
   - Document how wa_hca_eligibility relates to the broader eligibility table

   SQL Expressions:
   - Active member count by plan type
   - WA Medicaid member count by county for the latest file month
   - Month-over-month net enrollment change

   SQL Queries:
   - Top 5 WA counties by active Medicaid membership
   - Plan type distribution for currently active members
   - New enrollments vs lapses over the last 6 months
```

---

## Step 4 — Test with Business Questions

> Use these questions to validate the Genie Space works correctly after Step 3.

**Member Enrollment & Eligibility**
- How many members are actively enrolled right now, and what's the plan type breakdown?
- Which WA counties are seeing the fastest growth in Medicaid enrollment month over month?

**Appeals Operations**
- Are overturn rates getting better or worse month over month?
- Which denial reasons are we losing most often on appeal?
- How long does it take to resolve a pending appeal?
- Why are appeals with documentation taking longer to resolve than those without?

**Provider Activity**
- Which providers are submitting the most claims?
- Which providers have the worst payment-to-billed ratio?
- Do certain provider types drive a disproportionate share of our denials?
