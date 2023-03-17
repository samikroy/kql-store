_SentinelAudit()
| where SentinelResourceType =="Analytic Rule" and Description == "Create or update analytics rule."
| extend SentinelResourceId = tostring(ExtendedProperties.ResourceId)
| project TimeGenerated, SentinelResourceName, Status, Description, SentinelResourceKind, ExtendedProperties
| extend query_ = tostring(parse_json(tostring(parse_json(tostring(ExtendedProperties.UpdatedResourceState)).properties)).query)
| extend CallerName_ = tostring(ExtendedProperties.CallerName)
| extend CallerIpAddress_ = tostring(ExtendedProperties.CallerIpAddress)
| summarize arg_max(TimeGenerated,*) by query_, CallerIpAddress_, CallerName_, SentinelResourceName
