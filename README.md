# Python-Data-Visualization-Integration
Sprint covering the full Python visualization stack — from raw database connections to interactive dashboards and Power BI integration. Each library was selected for a specific role in the data workflow, and together they reflect how visualization decisions are made in practice: the right tool for the right audience, at the right stage of analysis.

---

## Notebook 1 — MySQL Connection

**File:** [`nivel1_mysql_connection.ipynb`](data/nivel1_mysql_connection.ipynb)

Before any visualization can happen, data needs to move. This notebook establishes the Python-to-MySQL pipeline using `mysql-connector-python`, covering connection management, cursor-based queries, and the more practical `pd.read_sql()` pattern for loading query results directly into a DataFrame.

A cursor is the interface between Python and the database engine — it sends queries and manages the result set, but returns raw tuples. `pd.read_sql()` wraps that process entirely, returning a structured DataFrame with column names intact, which is almost always preferable for analytical work. The distinction matters in production: cursors are appropriate when you need fine-grained control or are running DDL statements; `pd.read_sql()` is the default for any SELECT intended for analysis.

Connection hygiene is non-negotiable. Leaving database connections open in long-running scripts or notebooks exhausts the connection pool and causes silent failures downstream. Closing both cursor and connection explicitly — or using context managers — is standard practice.

---

## Notebook 2 — Matplotlib

**File:** [`nivel2_matplotlib.ipynb`](data/nivel2_matplotlib.ipynb)


Matplotlib is the foundation of Python visualization: verbose by design, but offering complete control over every element of a chart. The core mental model is the separation between `fig` (the canvas) and `ax` (the coordinate system where data is actually plotted). Every customization — titles, labels, colors, tick formatting — operates on the `ax` object, which makes multi-plot layouts predictable and composable.

The sprint covered bar charts, horizontal bars, scatter plots, histograms, and a 2×2 subplot dashboard — each chosen to address a specific analytical question rather than demonstrate chart types for their own sake.

The histogram versus bar chart distinction is worth stating clearly: bar charts compare discrete categories; histograms show the distribution of continuous values. Using a bar chart on continuous data would misrepresent the underlying structure. The `alpha` parameter controls opacity, which becomes important when plotting overlapping elements — lower values reveal density where data points coincide.

The scatter plot included a `axhline` break-even reference, a pattern that recurs throughout the sprint: anchoring visual context to a business threshold rather than letting the chart speak abstractly.

---

## Notebook 3 — Seaborn

**File:** [`nivel3_seaborn.ipynb`](data/nivel3_seaborn.ipynb)


Seaborn sits on top of Matplotlib but shifts the focus from rendering mechanics to statistical insight. Where Matplotlib requires explicit setup for every visual element, Seaborn infers sensible defaults and builds statistical context directly into the chart type.

The sprint used four chart types, each selected for a specific analytical purpose. Boxplots show the full distribution of a variable across categories — median, interquartile range, and outliers — in a single compact view. Violin plots extend this by revealing the shape of the distribution, which a boxplot cannot. When the question is not just "where is the center?" but "how is the data spread within each segment?", a violin plot surfaces information a boxplot would hide.

The correlation heatmap answered a specific question: does discounting drive sales volume in a way that damages margins? The matrix revealed the relationship between Discounts, Sales, Profit, and Profit Margin % simultaneously, which would require multiple separate charts in Matplotlib.

`sns.set_theme()` applies a consistent visual baseline across all charts in a session — grid style, font scaling, and palette defaults — so that a notebook full of charts maintains visual coherence without manual configuration on each plot.

Seaborn is the right tool during exploratory analysis: fast to write, statistically expressive, and clean enough for internal documentation and technical stakeholders.

---

## Notebook 4 — Plotly

**File:** [`nivel4_plotly.ipynb`](data/nivel4_plotly.ipynb)


Plotly changes the question from "what does this chart show?" to "what can the user do with this chart?" Every chart produced with Plotly Express is interactive by default — hover tooltips, zoom, pan, and click-to-filter require no additional configuration.

The sprint covered five chart types plus a multi-panel dashboard built with `make_subplots`. The scatter plot included `hover_data` with Country and Product, which means a stakeholder can identify specific records without requesting a filtered export. The treemap demonstrated hierarchical navigation: a single chart allows drill-down from Segment to Country to Product, replacing what would otherwise be a cascade of filtered views.

The practical distinction between Plotly and Seaborn is audience and workflow stage. Seaborn belongs in the analysis phase — when the goal is understanding the data. Plotly belongs in the presentation phase — when the goal is enabling someone else to explore it. A Seaborn chart in a Jupyter notebook during an EDA session is appropriate; that same chart in a stakeholder presentation is a missed opportunity.

A treemap is useful specifically when the data has a meaningful hierarchy and relative size matters at every level. Sales broken down by Segment > Country > Product is a natural fit: the area of each rectangle encodes magnitude, the nesting encodes structure, and the interactivity makes the hierarchy navigable rather than overwhelming.

---

## Notebook 5 — Power BI & Python Integration

**File:** [`nivel5_powerbi_python.ipynb`](data/nivel5_powerbi_python.ipynb)


The final notebook bridges Python and Power BI, demonstrating two distinct integration patterns that serve different purposes in a production workflow.

The first pattern — Python as a data source via Get Data → Python Script — uses Python for what it does best: flexible aggregation, reshaping, and transformation that would be cumbersome in Power Query. The script runs once at load time, and the resulting DataFrame is imported as a table into the data model. This is ETL, not visualization: Python handles the computation, Power BI handles the presentation layer.

The second pattern — Python visuals embedded in the report canvas — renders Matplotlib or Seaborn charts directly inside a Power BI report. The key mechanism is the `dataset` variable: Power BI automatically constructs a DataFrame from whichever fields are dragged into the visual's field well, and makes it available to the Python script as `dataset`. The script does not load data — it receives it, already filtered and scoped to the current report context.

This pattern has meaningful limitations. Python visuals are static — they render as images and do not support hover, drill-through, or cross-filtering with other visuals on the page. They also require Python to be installed and configured on the machine running the report, with all required packages present in the configured environment. For reports intended for broad distribution or embedded in the Power BI service, these constraints matter. For internal technical reports where statistical chart types not available natively in Power BI are needed, Python visuals fill a genuine gap.

The Power BI Embedded integration via `powerbiclient` (Exercise 3) requires a Pro or Premium license and was documented but not executed in this sprint.

---

## Library Decision Framework

The three visualization libraries address different stages of the analytical workflow:

**Matplotlib** is the right choice when precise control over chart layout, annotation, or export format is required — publication-quality figures, custom subplot arrangements, or charts embedded in automated reports.

**Seaborn** is the right choice during exploratory analysis and for communicating statistical structure to technical audiences. Less setup, more inference, and built-in support for distributions, correlations, and categorical comparisons.

**Plotly** is the right choice when the output will be presented to stakeholders who need to explore the data themselves — dashboards, presentations, or any context where a static image creates a bottleneck.

In practice, these tools are not mutually exclusive. A typical workflow uses Seaborn during EDA, Matplotlib for any custom static output, and Plotly for the final deliverable.
