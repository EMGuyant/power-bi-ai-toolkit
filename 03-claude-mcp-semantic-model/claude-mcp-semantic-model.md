# Power BI AI Development: Fix a Broken Model with Claude

This walkthrough connects Claude Desktop to a Power BI semantic model through Microsoft's Power BI MCP server, then runs the same working loop across five use cases on a model with broken measures. Every change is planned first, approved in Claude Desktop, applied in Power BI Desktop, and verified with a DAX query.

> **Disclaimer:** You're responsible for making sure your use of Claude follows your organization's AI usage and data policies. In this walkthrough, model metadata, DAX expressions, and query results are sent to Anthropic as part of the conversation, so confirm those policies allow it before you connect Claude to a model with your organization's data.

⚡ **Quick Links**

- 📖 Read the full blog post: [How to Use Claude Desktop for Power BI AI Development](https://ethanguyant.com/2026/11/16/how-to-use-claude-desktop-for-power-bi-ai-development/)
- 📁 Sample project: [`claude-powerbi-mcp`](./sample-file/)

---

## The Scenario

The sample is the sales model from earlier in the series, saved as a Power BI project (PBIP) with problems built in:

- The `DateTable` is deleted, so 23 measures fail and the Sales Overview page opens with broken visuals
- `Total Sales` uses the format string `0`, with no currency symbol or thousands separator
- Five product measures don't follow the naming pattern the others use
- None of the 34 measures in `_Measures` has a description

The semantic model includes the following tables:

- **Sales**: transactional sales data including Amount and SalesDate
- **Products**: product categories including Product
- **Regions**: regional breakdown including Region
- **Employee**: employee data including EmployeeID and Image
- **_Measures**: the report's measures, including a `Current Date` measure pinned to September 30, 2026 so the This Year and Last Year labels match the sample data

---

## Setup

1. In Claude Desktop, open **Settings** > **Developer** > **Edit Config** and add the MCP server.
2. Fully quit Claude Desktop, reopen it, and confirm the server is listed under **+** > **Connectors** in a new chat.
3. In Power BI Desktop, select **File** > **Options and settings** > **Options** > **Preview features**, turn on **Detect and reload external PBIP changes**, and restart Power BI Desktop.
4. On first use, review the server's EULA, tell Claude you accept it, and approve the `accept_eula` call.

To explore without changing anything, add `"--readonly"` to `args` in the config. It blocks every write until you remove it.

---

## The Working Loop

| Step | What you do |
|---|---|
| Connect | Open the PBIP folder connection for reads and edits, and the Power BI Desktop connection for DAX queries |
| Inspect | Ask Claude to summarize the model and flag issues before changing anything |
| Propose | Ask for a plan that names every object to change and what won't change |
| Approve | Approve each tool call in Claude Desktop, and compare it with the plan |
| Apply | Select **Apply external changes** in Power BI Desktop after Claude exports |
| Verify | Confirm the result with a DAX query through the Desktop connection, then check the report |

---

## Prompts & Results

### 0.0 Connect

Check for leftover connections, then open both.

> List all Power BI connections

> Open semantic model from PBIP folder 'C:\Projects\\<your-project-path>'.

> Connect to \<your-project-report> in Power BI Desktop for DAX queries only.

The folder connection loads the model's definition but no data. A DAX query against it returns "DAX query operations are not supported on offline connections," which is why the Desktop connection is needed.

### 1.0 Inspect the Model

> Inspect the model and provide a summary of tables, relationships, and measures.
> Then flag any issues you identify.

**Result:** 17 issues ranked from critical to hygiene. The critical one is the missing `DateTable`, confirmed with a DAX error from the live model.

**What to check:** Treat the report as findings to confirm. Each item names the measure or setting involved.

### 2.0 One Pass Through the Loop: Fix a Format String

> Plan a fix for the Total Sales format string. It currently uses 0, with no currency symbol or thousands separator.
>
> List every object you'll change, with its current and new value.
> Don't change anything else, and don't make changes until I approve the plan.
> Make the change through the PBIP folder connection only, then export it to the TMDL files.

After approving and applying:

> I've applied the changes in Power BI Desktop. Verify the Total Sales format string with a DAX query through the Desktop connection.

**Result:** One change to one object, from `0` to `\$#,0.00;(\$#,0.00);\$#,0.00`. Verified with `INFO.MEASURES()`.

**What to check:** Read the queries, not just the answer. `INFO.VIEW.MEASURES()` returned a null format string, and `INFO.MEASURES()` returned the new value.

### 3.0 Rebuild the Date Table

> Plan how to restore the DateTable the existing measures expect. Include every column the measures reference, the relationship to Sales[SalesDate], and marking it as a date table.
>
> List every object you'll create or modify, and show the table's DAX.
> Don't change any existing measures, and don't make changes until I approve the plan.
>
> Make changes through the PBIP folder connection only, then export them to the TMDL files.

Answer Claude's open question when you approve:

> Approved.
>
> For Year Category, use "This Year" for the current year, "Last Year" for the year before, and the year number for the rest.

> I've applied the changes in Power BI Desktop. Through the Desktop connection, verify that the Sales Metric (CY) and Sales Metric (LY) measures return the right values for each year, and check which of the broken measures now evaluate.

**Result:** All 23 measures evaluate, and Sales Overview shows $650.90K this year against $626.20K last year. The table Claude created:

```dax
DateTable =
VAR _minYear     = YEAR(MIN(Sales[SalesDate]))
VAR _maxYear     = YEAR(MAX(Sales[SalesDate]))
VAR _currentYear = YEAR([Current Date])
RETURN
    ADDCOLUMNS(
        CALENDAR(DATE(_minYear, 1, 1), DATE(_maxYear, 12, 31)),
        "Year", YEAR([Date]),
        "Year Category",
            SWITCH(
                YEAR([Date]),
                _currentYear,     "This Year",
                _currentYear - 1, "Last Year",
                FORMAT(YEAR([Date]), "0")
            )
    )
```

**What to check:** The plan's file list. Under source control, only `DateTable.tmdl`, `relationships.tmdl`, and `model.tmdl` should change in substance.

### 4.0 Understand a Model You Inherited

Attach a screenshot of the Top Performers cards to the prompt.

> Explain the Top Performers measure: what it calculates, how it builds its output, and any issues with the logic.
>
> In the report, it appears on one card per region, filtered to that region.
> Test each issue with DAX queries through the Desktop connection, using the measure itself, and say whether it affects the measure in that context. Don't change anything.

**Result:** Correct on every region card across all 15 region and year contexts. Of seven issues, one matters outside that context: the measure ranks region-employee pairs, not employees, so it's only correct when filtered to a single region.

**What to check:** Give Claude the context the measure is used in. Without it, seven issues read like seven fixes.

### 5.0 Document the Model in Bulk

> Plan descriptions for every measure in _Measures. Each description should say what the measure returns and any filter behavior a report author needs to know, in one or two sentences of plain business language.
>
> For Top Performers, note that it ranks region-employee pairs, so it's only correct when filtered to a single region.
>
> Show the proposed descriptions as a table I can review. Don't change anything else, and don't make changes until I approve. Make changes through the PBIP folder connection only, then export them to the TMDL files.

> Approved. Update and export the descriptions.

> I've applied the changes in Power BI Desktop. Through the Desktop connection, confirm every measure in _Measures has a description, and list any that don't.

**Result:** 34 descriptions in one approved batch, stored as `///` description lines in `_Measures.tmdl` and shown as tooltips in the **Data** pane.

**What to check:** Spot-check the descriptions that make claims about filter behavior against the DAX.

### 6.0 Refactor at Scale

> Plan how to rename these five measures to match the naming pattern the other product measures use: Smartphone CY Sales, Headphones_Sales_Metric_CY, laptop_sales_cy, tblt sales total, TV Sales (Total).
> Find every measure that references them, and include those reference updates in the plan.
>
> Show a change log with each old name, new name and dependent object. Don't change anything else, and don't make changes until I approve.
> Make changes through the PBIP folder connection only, then export them to the TMDL files.

> I've applied the changes in Power BI Desktop. Through the Desktop connection, confirm the five measures exist under their new names, the old names are gone, every measure that referenced them still evaluates, and provide a list of affected visuals.

> Update the impacted visuals.

**Result:** Five measures renamed, no dependent measures. One visual broke, as Claude warned in the plan. Claude fixed it by replacing 20 references in its `visual.json`, using Claude Desktop's folder access, not the MCP server.

**What to check:** Renaming in the model files doesn't update report visuals. Plan for the report side before you approve.

### 7.0 Plan a Report Page

Attach a screenshot of the Sales Overview page to the prompt.

> I want to add a report page that answers: how is each salesperson performing this year compared with last year, by region and by product?
>
> Plan the page: the visuals, the fields each one uses, filters and slicers, and the layout. A screenshot of the existing Sales Overview page is attached so the new page can match its style.
> Use existing measures where they fit. List any new measures the page needs, with their DAX, format strings and descriptions.
>
> Don't create anything yet, and don't edit any report files. When I approve, create only the new measures through the PBIP folder connection, then export them to the TMDL files.

**Result:** A page plan with two matrices, six new measures in a `Sales\Salesperson` display folder, and a mock-up rendered in Claude Desktop. Verification found that `Salespeople Growing YoY` returned 0 instead of blank for employees with no sales. The fixed measure:

```dax
Salespeople Growing YoY =
VAR _priorYearExists =
    NOT ISBLANK(
        CALCULATE([Sales Metric (LY)], ALLEXCEPT(Sales, DateTable))
    )
VAR _emp =
    ADDCOLUMNS(VALUES(Employee[EmployeeID]), "@yoy", [Sales Metric YOY])
RETURN
    IF(
        _priorYearExists
            && NOT(ISBLANK([Sales Metric (CY)]) && ISBLANK([Sales Metric (LY)])),
        COUNTROWS(FILTER(_emp, [@yoy] > 0)) + 0
    )
```

**What to check:** The mock-up is a design reference, not a Power BI page. Confirm its numbers with a query before you build.

---

## Prompt Patterns That Work

| Pattern | Example |
|---|---|
| Plan before changing | "Don't make changes until I approve the plan." |
| Name what won't change | "Don't change any existing measures." |
| Name the connection | "Make changes through the PBIP folder connection only, then export them to the TMDL files." |
| Verify through the live model | "Through the Desktop connection, verify that..." |
| Give usage context | "In the report, it appears on one card per region, filtered to that region." |

---

## Guardrails

- **Back up first.** Keep the project in Git. Each export rewrites every file in the definition folder, so `git diff` shows what actually changed.
- **Set tool permissions by risk.** In **Settings** > **Connectors** > **powerbi-modeling-mcp**, set the read-only tools to **Always allow** and leave the write/delete tools on **Needs approval**.
- **Know what leaves your machine.** Metadata, DAX expressions, and query results go into the conversation and are sent to Anthropic.

---

## 🛠 How to Use

1. **Clone** this repository.
2. **Open** the `.pbip` file in [`sample-file`](./sample-file/) in Power BI Desktop. The project doesn't include cached data, so it opens without data.
3. **Point** the data sources at the CSV files in [`sample-data`](./sample-data/): **Transform data** > **Data source settings** > **Change Source**, then **Refresh**.
4. **Save** the project, then work through the prompts above in order. The use cases build on each other, starting with the date table.

To start over, discard your changes with `git checkout -- 03-claude-mcp-semantic-model/sample-file`.

---

## 🔑 Prerequisites

To follow along with this post, you need:

- **Claude Desktop** on Windows, with a Claude plan
- **Node.js**, so `npx` can run the MCP server
- **Power BI Desktop**, August 2026 release or later, with the **Detect and reload external PBIP changes** preview feature turned on
- **No Fabric capacity or Copilot license.** The local MCP server works against Power BI Desktop and PBIP files.

As of November 2026, the walkthrough uses Power BI Authoring MCP server 1.0.0. See [Power BI Authoring MCP server](https://learn.microsoft.com/power-bi/developer/mcp/power-bi-authoring-mcp) on Microsoft Learn and the [powerbi-modeling-mcp repository](https://github.com/microsoft/powerbi-modeling-mcp).

---

> **Disclaimer:** Claude's responses vary from run to run, so the plans, issue lists, and wording you get won't match the ones shown here exactly. The pattern should hold. Check every change against your own model before you rely on it.
