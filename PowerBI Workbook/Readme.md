



#### Overview

- Activity Over Time
```
PowerBIActivity
| summarize count() by Activity, bin(TimeGenerated,{TimeRange:grain})
```
- Events Occured
```
PowerBIActivity
|summarize  count() by Activity
```
- Workspace Used
```
PowerBIActivity
|summarize  count() by PbiWorkspaceName
```
- Report Accessed
```
PowerBIActivity
|summarize  count() by ReportName
```
- Dataset Accessed
```
PowerBIActivity
|summarize  count() by DatasetName
```
- Activity by DistributionMethod
```
PowerBIActivity
| summarize count() by DistributionMethod
```
- Activity through UserAgent
```
PowerBIActivity
| summarize count() by UserAgent
```
#### Datasets

- Dataset Directly Accessed
```
PowerBIActivity
| where isempty(ReportName) and isnotempty(DatasetName)
|summarize  count() by DatasetName
```
- Activity Directly on Datastes
```
PowerBIActivity
| where isempty(ReportName) and isnotempty(DatasetName)
| summarize count() by Activity
```
- Direct Datasets Activities
```
let PbiTrend = (PowerBIActivity
| where isnotempty(DatasetName)
| make-series Trend = count() default = 0 on TimeGenerated in range({TimeRange:start}, {TimeRange:end}, {TimeRange:grain}) by DatasetName);
let PbiSummary = (PowerBIActivity
| where isnotempty(DatasetName)
|summarize TotalActivity = count(), CreateDataset = countif(Activity =="CreateDataset"),UpdateDatasetParameters
 = countif(Activity =="UpdateDatasetParameters") by DatasetName);
PbiTrend
| join kind=inner(
PbiSummary
) on DatasetName
```
#### Reports

- Activities Across Reports
```
let PbiTrend = (PowerBIActivity
| where isnotempty(DatasetName) and  isnotempty(ReportName)
| make-series Trend = count() default = 0 on TimeGenerated in range({TimeRange:start}, {TimeRange:end}, {TimeRange:grain}) by ReportName, DatasetName);
let PbiSummary = (PowerBIActivity
| where isnotempty(DatasetName) and  isnotempty(ReportName)
|summarize TotalActivity = count(), ViewReport = countif(Activity =="ViewReport"),CreateReport = countif(Activity =="CreateReport") by ReportName, DatasetName);
PbiTrend
| join kind=inner(
PbiSummary
) on ReportName,DatasetName
```
#### IP and User Activity Trend
- IP Address Activity Trend
```
let PbiTrend = (PowerBIActivity
| make-series Trend = count() default = 0 on TimeGenerated in range({TimeRange:start}, {TimeRange:end}, {TimeRange:grain}) by SrcIpAddr);
let PbiSummary = (PowerBIActivity
|summarize count() by SrcIpAddr);
PbiTrend
| join kind=inner(
PbiSummary
) on SrcIpAddr
```
- User Activity Trend
```
let PbiTrend = (PowerBIActivity
| make-series Trend = count() default = 0 on TimeGenerated in range({TimeRange:start}, {TimeRange:end}, {TimeRange:grain}) by ActorName);
let PbiSummary = (PowerBIActivity
|summarize count() by ActorName);
PbiTrend
| join kind=inner(
PbiSummary
) on ActorName
```
- User Activity Across IP Addresses
```
PowerBIActivity
|summarize  count() by ActorName, SrcIpAddr
```



