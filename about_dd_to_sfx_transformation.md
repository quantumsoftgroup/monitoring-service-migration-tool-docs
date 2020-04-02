# Datadog to SignalFx

This application allows you to convert Datadog dashboards and monitors into SignalFx 
dashboards and detectors.
Below you will find a description of what and how is converted from one system to another, 
and what cannot be converted.

# Dashboard Widgets transformation
There are several types of widgets in DataDog. A list of them can be found
[on this page](https://docs.datadoghq.com/dashboards/widgets/).

### Supported transformations 
1. <b>Free Text</b> and <b>Note</b>. Contains text. Transforms into charts with `Text chart` type. 
Widgets with this type are also used to display errors that occurred during the transformation. 
1. <b>Query Value</b>. Query values display the current value of a given metric. Transforms into `Single value chart` type.
1. <b>Time Series</b> and <b>Heat Map</b>. Allows you to display the evolution of one or more metrics over time. 
Transforms into `Time Series chart` type. 
1. <b>Top List</b>. Allows you to display a list of Tag values like hostname or service with the most or least of any metric value. Transforms into `List chart` type with `top()` method.

All other DataDog widget types don't exist in SignalFx. If you will try to transform a widget with an unsupported type, a text widget with an error message will be created instead.

Transformation includes functions and methods transformation, visualization transformation (colors, resolution) etc.

### Function transformation
Certain functions that can be used in DD widget queries doesn't exist or have analogs in SFX. 
Widgets containing such queries won't be automatically exported to SFX and will be converted to Text widget instead (used as a placeholder). 
Here is the list of these functions:
1. anomalies
1. outliers
1. forecast
1. log2
1. count_nonzero
1. count_not_null	
1. fill
1. top
1. fill
1. monotonic_diff
1. robust_trend
1. trend_line
1. piecewise_constant
1. rollup
1. autosmooth
1. ewma
1. median
1. timeshift
# Monitors transformation
This application allows to transform query monitors into SFX Detectors. Composite and Service Check alerts are unsupported because they don't exist in SignalFX.

# Export
The tool can export results of transformations into Terraform (0.11) configuration files or load it durectly to SignalFx via API
