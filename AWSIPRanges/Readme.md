
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
