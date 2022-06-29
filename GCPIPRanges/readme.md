# In KQL Using at times we might need the IP Address for GCP IP Address ranges.

Here is an azure function developped in KQL which can be used to fetched the IP range for GCP

Query

```
externaldata(ip_prefix: string)
[ 
   h@'https://www.gstatic.com/ipranges/cloud.json'
]
with(format='multijson', ingestionMapping='[{"Column":"ip_prefix","Properties":{"Path":"$.prefixes"}},{"Column":"ipv4Prefix","Properties":{"Path":"$.prefixes.ipv4Prefix"}}]')
| mv-expand todynamic(ip_prefix)
| extend ipv4Prefix_ = tostring(ip_prefix.ipv4Prefix)
| extend scope_ = tostring(ip_prefix.scope)
| extend service_ = tostring(ip_prefix.service)
| project-away ip_prefix

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
          "name": "GetGCPIPRanges",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "GetGCPIPRanges",
            "category": "Security",
            "FunctionAlias": "GetGCPIPRanges",
            "query": "externaldata(ip_prefix: string)[ h@'https://www.gstatic.com/ipranges/cloud.json']with(format='multijson', ingestionMapping='[{\"Column\":\"ip_prefix\",\"Properties\":{\"Path\":\"$.prefixes\"}},{\"Column\":\"ipv4Prefix\",\"Properties\":{\"Path\":\"$.prefixes.ipv4Prefix\"}}]')| mv-expand todynamic(ip_prefix)| extend ipv4Prefix_ = tostring(ip_prefix.ipv4Prefix)| extend scope_ = tostring(ip_prefix.scope)| extend service_ = tostring(ip_prefix.service)| project-away ip_prefix",
            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FGCPIPRanges%2Fgcpiprangesarmtemplate.json)


