# NewRelic to SignalFx
This application allows you to convert SignalFX dashboards and alert detectors into New Relic 
dashboards and alerts.
Below you will find a description of what and how is converted from one system to another, 
and what cannot be converted.

## Dashboard Chart Types transformation
There are several types of charts in New Relic. A list of them can be found
[on this page](https://docs.newrelic.com/docs/insights/insights-api/manage-dashboards/insights-dashboard-api) in the `Supported visualizations` section.<br />

### Supported transformations from New Relic to SignalFX
All types of New Relic widget visualizations can be divided into groups, depending on how they are transform into sfx charts.
1. <b>Note</b>. Contains plain text. Transforms from charts with `markdown` visualisation. 
Widgets with this type are also used to display errors that occurred during the transformation. 
For example, if you try to transform a widget with an unsupported type, a text widget with an error message will be created instead.
1. <b>Timeseries Line Chart</b>. Return data as a time series broken out by a specified period of time. Contains one or more plots. 
Transforms from charts based by NRQL-query with `TIMESERIES` component used in query. Those charts can be with visualisation type from this list:<br />
`line_chart`, `faceted_line_chart`, `comparison_line_chart`, `faceted_area_chart`.<br />
Some charts in New Relic can be created without using the nrql language. These are Charts with the visualization type `metric_line_chart`.
They transform into Timeseries Line Charts too.
1. <b>Single value Chart</b>. Single value charts show a single value for a datapoint as it changes over time. Contain exactly one plot.
Transforms from charts based by NRQL-query without `TIMESERIES` component used in query, that have only one datapoint.
1. <b>List Chart</b>. List charts are similar to single value charts, but they display multiple datapoints at each point in time. Contain two or more plots. 
Transforms from charts based by NRQL-query without `TIMESERIES` component used in query.
List of visualizations, which will be transformed into a list:<br />
`facet_table`, `attribute_sheet`,`billboard_comparison`, `bar_chart`, `pie_chart`, `gauge`, `billboard`.

### Unsupported transformations
Here is a list of visualization types whose transformation is not supported:
1. heatmap
1. histogram
1. event_table
1. funnel
1. json
1. list

## NRQL Components Transformations
NRQL - NRQL is a query language you can use to query the New Relic database. Most of New Relic charts (widgets) are made using NRQL.<br />
You can read about NRQL syntax [here](https://docs.newrelic.com/docs/query-data/nrql-new-relic-query-language/getting-started/nrql-syntax-components-functions)<br />
Below are explanations of how nrql-components are used to transform charts from New Relic to SignalFX.<br />

### SELECT
SELECT expression may contain single attribute, or function with attributes:<br />
1. SELECT attribute ...<br />
1. SELECT function(attribute) ...<br />
In first case we transform attribute as metric<br />
In second case we transform attribute as metric, and function as [SignalFlow method](https://developers.signalfx.com/signalflow_analytics/methods/above_stream_method.html). 
At the moment, not all functions can be transformed. List of supported functions: average, count, max, median, min, percentile, stddev, sum.

### Function transformation
Certain functions that can be used in NRQL widget queries doesn't exist or have analogs in SFX. 
Widgets containing such queries won't be automatically exported to SFX and will be converted to Text widget instead (used as a placeholder). 
Here is the list of these functions:
1. apdex
1. buckets
1. eventType
1. filter
1. funnel
1. histogram
1. keyset
1. latest
1. percentage
1. rate
1. uniqueCount
1. uniques

### FROM
FROM is used to specify [data type](https://docs.newrelic.com/docs/query-data/nrql-new-relic-query-language/getting-started/introduction-nrql#what-you-can-query) divided into groups. 
In SFX, metrics are not grouped, so transformation is not provided.

### WHERE
WHERE clauses is used to filter results. WHERE clause contain attribute (or function), operator, and value to compare. Example looks like:<br />
```bash
SELECT * FROM SystemSample WHERE function(param) = 10
```
operators list is: `=, !=, <, <=, >, >=, IS NULL, IS NOT NULL, IN, NOT IN, LIKE, NOT LIKE`<br />
Currently transformation is available only for those operations: `=, !=, IN, NOT IN`. Transformation makes filter for each WHERE case for each plot in chart

### FACET CASES
FACET CASES contains coma-separated list of WHERE clauses. Each clause, independently of the others, is applied to one plot. 
Thus, if we have a chart that contains 2 plots, then when adding a FACET CASE with two WHERE clauses, 
the primary two plots are duplicated for each of them. Finally, we get 4 plots, 2 of which contain the conditions of one WHERE clause, and 2 of the other.
Thus, the transformation process is made - the initial graphs are replaced by graphs with filters for each WHERE clause.

### SINCE and UNTIL
Defines the beginning and end point of time range. Both absolute and relative values can be used.
For [ABSOLUTE time range](https://docs.newrelic.com/docs/insights/use-insights-ui/time-settings/set-time-range-insights-dashboards-charts#)
(since TIME_A until TIME_B) transformation sets Custom absolute time range to sfx chart<br />
For [RELATIVE time range](https://docs.newrelic.com/docs/insights/use-insights-ui/time-settings/set-time-range-insights-dashboards-charts#relative-range)
transformation only supports case `SINCE integet units AGO`. Other cases is not possible to transform.

### WITH TIMEZONE
Defines timezone for chart. The transformation simply sets the desired time zone when creating the chart as parameter.

### COMPARE WITH
COMPARE WITH makes copy for each plot, but with another time range. This allows you to compare current metric values with previous ones.
Transformation makes copy for each plot, and apply [timeshift method](https://developers.signalfx.com/signalflow_analytics/methods/timeshift_stream_method.html) to the copies.

### FACET
FACET component is used to separating and grouping results by attribute values.
Transformation applies `top()` method for plots.

### LIMIT
LIMIT indicates the maximum number of records that can be displayed on the widget when using the FACET component. 
Transforms into the `count` attribute for the `top()` method.

### TIMESERIES
Defines charts with timeseries. Transformation of TIMESERIES values is not supported yet (auto scale by default)

### EXTRAPOLATE
Transformation for this nrql-component is not supported yet

## Alert Policies transformation
Alert policy is a group of Alert Conditions. Each condition defines a threshold that triggers an alert.
Alert conditions are divided into types by the product they use. Currently, a transformation for the Infrastructure product is supported.
Each condition transforms into SignalFX Alert Detector.
