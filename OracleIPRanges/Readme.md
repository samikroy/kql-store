
## In KQL Using at times we might need the IP Address for Oracle IP Address ranges.

Here is an azure function developped in KQL which can be used to fetched the IP range for Oracle

Query


```
externaldata(Region:string , CIDRS: string )
[ 
   h@'https://raw.githubusercontent.com/samikroy/kql-store/main/OracleIPRanges/oracleipranges.json'
]
with(format='multijson', ingestionMapping='[{"Column":"Region","Properties":{"Path":"$.region"}},{"Column":"CIDRS","Properties":{"Path":"$.cidrs"}}]')
| mv-expand todynamic(CIDRS)
| extend CIDR = CIDRS.cidr
| extend Tags = CIDRS.tags
| mv-expand Tags
| project Region, CIDR, Tags


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
          "name": "GetOracleIPRanges",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "GetOracleIPRanges",
            "category": "Security",
            "FunctionAlias": "GetOracleIPRanges",
            "query": "externaldata(Region:string , CIDRS: string )[h@'https://raw.githubusercontent.com/samikroy/kql-store/main/OracleIPRanges/oracleipranges.json'] with(format='multijson', ingestionMapping='[{"Column":"Region","Properties":{"Path":"$.region"}},{"Column":"CIDRS","Properties":{"Path":"$.cidrs"}}]') | mv-expand todynamic(CIDRS) | extend CIDR = CIDRS.cidr | extend Tags = CIDRS.tags | mv-expand Tags | project Region, CIDR, Tags",
            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FOracleIPRanges%2Foracleiprangesarmtemplate.json)
