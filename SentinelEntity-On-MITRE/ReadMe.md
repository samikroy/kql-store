## Sentinel Entities on MITRE

This query will help you visualize the MITRE techniques tagged with each type of Entity


Query

```
SecurityAlert
| where TimeGenerated > ago(14d)
| summarize arg_max(TimeGenerated,*) by SystemAlertId
| where isnotempty(Tactics)
| extend Tactics = split(Tactics,',')
| mv-expand Tactics
| extend Tactics = tostring(Tactics)
| mv-expand todynamic(Entities)
| extend EntityType = tostring(Entities.Type)
| summarize count() by Tactics, EntityType
| render columnchart 

```

Visual
