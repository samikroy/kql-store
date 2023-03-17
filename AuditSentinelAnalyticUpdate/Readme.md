



## In KQL Using at times we might need find ourt who changed a detection rule or waht changes were done.
Here is an azure function developped in KQL which can be used to display the changes.

Query


```

_SentinelAudit()
| where SentinelResourceType =="Analytic Rule" and Description == "Create or update analytics rule."
| extend SentinelResourceId = tostring(ExtendedProperties.ResourceId)
| project TimeGenerated, SentinelResourceName, Status, Description, SentinelResourceKind, ExtendedProperties
| extend query_ = tostring(parse_json(tostring(parse_json(tostring(ExtendedProperties.UpdatedResourceState)).properties)).query)
| extend CallerName_ = tostring(ExtendedProperties.CallerName)
| extend CallerIpAddress_ = tostring(ExtendedProperties.CallerIpAddress)
| summarize arg_max(TimeGenerated,*) by query_, CallerIpAddress_, CallerName_, SentinelResourceName

```

Now, while we can use this query in our KQL queries and then it will also be useful to have this as a deployable template.

Here is the code for ARM Template

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspaceName": {
      "type": "String"
    },
    "location": {
      "type": "String"
    }
  },
  "resources": [
    {
      "type": "Microsoft.OperationalInsights/workspaces",
      "apiVersion": "2017-03-15-preview",
      "name": "[parameters('workspaceName')]",
      "location": "[parameters('location')]",
      "resources": [
        {
          "type": "savedSearches",
          "apiVersion": "2020-08-01",
          "name": "_AuditSentinelAnalytics",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "_AuditSentinelAnalytics",
            "category": "Security",
            "FunctionAlias": "_AuditSentinelAnalytics",
            "query": "_SentinelAudit() | where SentinelResourceType =="Analytic Rule" and Description == "Create or update analytics rule." | extend SentinelResourceId = tostring(ExtendedProperties.ResourceId) | project TimeGenerated, SentinelResourceName, Status, Description, SentinelResourceKind, ExtendedProperties | extend query_ = tostring(parse_json(tostring(parse_json(tostring(ExtendedProperties.UpdatedResourceState)).properties)).query) | extend CallerName_ = tostring(ExtendedProperties.CallerName) | extend CallerIpAddress_ = tostring(ExtendedProperties.CallerIpAddress) | summarize arg_max(TimeGenerated,*) by query_, CallerIpAddress_, CallerName_, SentinelResourceName",            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FOracleIPRanges%2Foracleiprangesarmtemplate.json)
