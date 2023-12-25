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

Here is an azure function developped in KQL 

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
          "name": "SentinelEntityOnMITRE",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "SentinelEntityOnMITRE",
            "category": "Security",
            "FunctionAlias": "SentinelEntityOnMITRE",
            "query": "SecurityAlert| where TimeGenerated > ago(14d) | summarize arg_max(TimeGenerated,*) by SystemAlertId | where isnotempty(Tactics) | extend Tactics = split(Tactics,',') | mv-expand Tactics | extend Tactics = tostring(Tactics) | mv-expand todynamic(Entities) | extend EntityType = tostring(Entities.Type) | summarize count() by Tactics, EntityType | render columnchart",
            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FSentinelEntity-On-MITRE%2FSentinelEntity-On-MITREarmtemplate.json)



