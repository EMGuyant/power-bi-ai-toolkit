# Build Better Reports Faster with Power BI Copilot in Power BI Desktop

In this post, we walk through using Power BI Copilot for report design across a realistic retail reporting scenario, covering page generation, prompt engineering, follow-up edits, and where Copilot hands off to the developer.

⚡ **Quick Links**

- 📖 Read the full blog post: [Power BI's AI Toolkit: Build Better Reports Faster with Copilot](https://ethanguyant.com/2026/06/02/power-bi-ai-toolkit-build-better-reports-faster-with-copilot/)
- 📁 Sample file: [`copilot-pbi-desktop-report`](./sample-file/copilot-powerbi-desktop.pbix)

---

## The Scenario

A Power BI developer at a mid-sized retail company has a clean, tested semantic model and a blank report canvas. The ask: three report pages for stakeholders by end of week. This walkthrough uses that scenario to explore what Power BI Copilot can realistically do at each stage of report development.

The semantic model includes the following tables:

- **Sales** — transactional sales data including Amount and SalesDate
- **Products** — product categories including Product
- **Regions** — regional breakdown including Region
- **Employee** — employee data including EmployeeID and Image
- **DateTable** — date intelligence table including Date, Year, Month, and Year Category

---

## Prompts & Results

### 1.0 Suggest Content: Regional Sales Analysis

Using the built-in **"Suggest content for this report"** option to generate a Regional Sales Analysis page as a baseline for what Copilot produces without a custom prompt.

- 🟡 **Yellow Light** — Reasonable starting point. Visual and field selection is functional but layout and formatting need manual work.

### 2.0 Custom Prompt: Product Performance Page

Using a business-question driven prompt to generate a more targeted Product Performance page.

**Prompt used:**
> "Create a product performance page that answers the question: which products are driving revenue this year and how are they trending compared to last year?"

- 🟡 **Yellow Light** — Strong initial output. KPI cards, YOY comparison, and regional breakdown generated without specifying visual types. One chart required a follow-up prompt to improve readability.

**Follow-up prompt:**
> "Change the Sales Metric (CY) and Sales Metric (LY) by Year and Region chart to a clustered bar chart."

> **Note:** When a visual has two measures on the y-axis, Power BI uses the legend to differentiate between those measures. Adding a category field such as Region to the legend is not supported in this configuration, either through prompting or manual editing in the Build pane. Changing the visual type to a clustered bar chart is the most practical workaround.

### 3.0 Narrative Summary Visual

Using the Copilot-powered **Narrative** visual to generate a plain-language summary of the Product Performance page.

- 🟡 **Yellow Light** — Functional output. Responds to slicer selections. Language is generic and benefits from follow-up prompts or manual editing.

> **Note:** Before generating a narrative, review the [Copilot Preview Terms of Use](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms) linked in the narrative pane. Particularly relevant for organizational or production report contexts.

---

## Prompt Patterns That Work

| Pattern | Example |
|---|---|
| Business question driven | "Create a page that answers the question: which products are driving revenue this year?" |
| Audience referenced | "Create a page for regional sales managers showing..." |
| Measure names specified | "Use Sales Metric (CY) and Sales Metric (LY)..." |
| Targeted follow-up | "Change the [visual title] chart to a clustered bar chart" |

---

## What Copilot Does Well

- Getting from blank canvas to a reviewable draft quickly
- Selecting relevant visuals and fields based on a business question
- Generating KPI cards, comparison charts, and slicers without explicit instruction
- Producing narrative summaries that respond to filter context

## Where Copilot Hands Off to the Developer

- Layout, spacing, and alignment on the canvas
- Formatting, theming, and visual titles
- Slicer placement and cross-filter behavior
- Structural edits to visuals with two measures on the y-axis
- Final measure and field validation in the Build pane

---

## 🛠 How to Use

1. **Clone** this repository to explore the prompts and sample file directly.
2. **Open** the provided sample dataset (Star Schema: Sales, Products, Employee, Regions, DateTable).
3. **Copy/Paste** the prompts into the Copilot pane in Power BI Desktop to see the generative results in real time.

---

## 🔑 Prerequisites

To follow along with this post, you require:

- **Fabric/Premium Capacity:** A paid Fabric capacity (F2+) or Power BI Premium (P1+)
- **Copilot Enabled:** Tenant-level admin approval for Copilot features. See [Enable Fabric Copilot for Power BI](https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-enable-power-bi)
- **Workspace Association:** Admin, member, or contributor access to a compatible workspace
- **Q&A Feature Switch:** Enabled on the semantic model

---

> **Disclaimer:** Because of the generative nature of Power BI Copilot, it may not produce the exact same pages, visuals, or field selections as shown here. The patterns and behaviors should be consistent, but the specific output may vary each time you run the same prompt.
