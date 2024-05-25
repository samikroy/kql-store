### Azure Service Health Workbook Queries

#### Use the below to deplot the workbook in your environment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FAzure%20Service%20Health%2FAzureServiceHealthWorkbookARM.json)

### Use below queries for know the specific details about the Open AI instances

####   Subscriptions Impacted
```
AzureActivity
| where CategoryValue == "ServiceHealth" 
| summarize TimeGenerated=arg_max(TimeGenerated,*) by SubscriptionId
| extend SubscriptionId == strcat('/subscriptions/', SubscriptionId)
```
#### Impacted Regions
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| extend ImpactedRegions = tostring(Properties_d.region)
| summarize count() by ImpactedRegions
```
####  Type of Incidents
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| extend IncidentType= tostring(Properties_d.incidentType)
| summarize count() by IncidentType
```
#### Service Health on Status
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| extend Status = tostring(Properties_d.activityStatusValue)
| summarize count() by Status
```
#### Service Health Across Services
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| where TrackingId in ({TrackingID_P:value}) or '{TrackingID_P:label}'=='All'
| extend Service = tostring(Properties_d.service)
| summarize count() by Service
```
#### Resolution Time
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| extend Service = tostring(Properties_d.service)
| extend ImpactMitigationTime = tostring(Properties_d.impactMitigationTime)
| extend ImpactStartTime = tostring(Properties_d.impactStartTime)
| extend ["ResolutionTime"]=datetime_diff('hour',todatetime(ImpactMitigationTime),todatetime(ImpactStartTime))
| extend ImpactedRegions = todynamic(parse_json(tostring(parse_json(tostring(parse_json(Properties).impactedServices))[0].ImpactedRegions)))
| mv-expand ImpactedRegions
| project  Service,ImpactedRegions.RegionName, ["ResolutionTime"], _ResourceId
```
#### Current Stage
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,*) by TrackingId
| where TrackingId in ({TrackingID_P:value}) or '{TrackingID_P:label}'=='All'
| extend Stage = tostring(parse_json(Properties).stage)
| summarize count() by Stage
```
#### Analysis of Incidents
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize arg_max(TimeGenerated,) by TrackingId
| extend IncidentType= tostring(Properties_d.incidentType)
| extend ImpactMitigationTime = tostring(Properties_d.impactMitigationTime)
| extend ImpactStartTime = tostring(Properties_d.impactStartTime)
| extend Status = tostring(Properties_d.activityStatusValue)
| extend Service = tostring(Properties_d.service)
| extend Title = tostring(Properties_d.defaultLanguageTitle)
| extend ImpactedRegions = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).impactedServices))[0].ImpactedRegions)))
| summarize ImpactedSubscriptions=make_set(SubscriptionId), TimeGenerated=arg_max(TimeGenerated,) by Level, ImpactedRegions, Service, TrackingId, Status
| project TimeGenerated
        , TrackingId
        , Title
        , Status
        , ["Impact Start Time"]=todatetime(ImpactStartTime)
        , Level
        , IncidentType
        , ImpactedSubscriptions
        , Service
        , ["# Subs."]=array_length(ImpactedSubscriptions)
        , ImpactedRegions
        , ["# Regions"]=array_length(parse_json(ImpactedRegions))
        , todatetime(ImpactMitigationTime)
        , ["ResolutionTime (hours)"]=datetime_diff('hour',todatetime(ImpactMitigationTime),todatetime(ImpactStartTime))
| order by TimeGenerated 
```
#### Affected Subscription
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| summarize ImpactedSubscriptions=make_set(SubscriptionId) by TrackingId
| project ImpactedSubscriptions, TrackingId
| mv-expand ImpactedSubscriptions
| project Subscription=strcat('/subscriptions/',ImpactedSubscriptions), TrackingId
| extend Info=strcat('https://app.azure.com/h/',TrackingId)
| order by TrackingId asc
```
#### Affected Regions
```
AzureActivity
| where CategoryValue == "ServiceHealth"
| extend TrackingId = tostring(Properties_d.trackingId)
| extend ImpactedRegions = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).impactedServices))[0].ImpactedRegions)))
| project todynamic(ImpactedRegions), TrackingId
| mv-expand ImpactedRegions
| project Region=strcat(tostring(ImpactedRegions.RegionName),' (',tostring(ImpactedRegions.RegionId),')'),TrackingId
| distinct Region, TrackingId
| extend Info=strcat('https://app.azure.com/h/', TrackingId)
| order by TrackingId asc

```

