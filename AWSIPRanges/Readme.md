
## In KQL Using at times we might need the IP Address for AWS IP Address ranges.

Here is an azure function developped in KQL which can be used to fetched the IP range for GCP

Query


```
externaldata(ip_prefix: string , region:string , service: string, network_border_group:string )
[ 
   h@'https://raw.githubusercontent.com/samikroy/kql-store/main/AWSIPRanges/awsipranges.json'
]
with(format='multijson', ingestionMapping='[{"Column":"ip_prefix","Properties":{"Path":"$.ip_prefix"}},{"Column":"region","Properties":{"Path":"$.region"}},{"Column":"network_border_group","Properties":{"Path":"$.network_border_group"}}]')

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
          "name": "GetAWSIPRanges",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "GetAWSIPRanges",
            "category": "Security",
            "FunctionAlias": "GetAWSIPRanges",
            "query": "externaldata(ip_prefix: string , region:string , service: string, network_border_group:string )[ h@'https://raw.githubusercontent.com/samikroy/kql-store/main/AWSIPRanges/awsipranges.json'] with(format='multijson', ingestionMapping='[{\"Column\":\"ip_prefix\",\"Properties\":{\"Path\":\"$.ip_prefix\"}},{\"Column\":\"region\",\"Properties\":{\"Path\":\"$.region\"}},{\"Column\":\"network_border_group\",\"Properties\":{\"Path\":\"$.network_border_group\"}}]')",
            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FAWSIPRanges%2Fawsiprangesarmtemplate.json)
