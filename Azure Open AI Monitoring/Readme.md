### Azure Open AI Monitoring Workbook Queries

#### Use the below to deplot the workbook in your environment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FAzure%2520Open%2520AI%2520Monitoring%2FAzureOpenAIWorkbookARM.json)

### Use below queries for know the specific details about the Open AI instances

####   Open AI Operations
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| summarize count() by OperationName
```
#### Open AI Operations
```
let data = (AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| summarize count() by ResultSignature
| extend Result = case (ResultSignature == 404, "Not found", ResultSignature == 200, "Success", "Unknown"));
let data1= (AzureDiagnostics
| summarize make_list(DurationMs) by ResultSignature);
data
| join kind=inner
(data1)
 on ResultSignature
```
####  Open AI Instances Activity
```
 AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| summarize count() by Resource
```
#### Open AI Total Duration 
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where Category == "RequestResponse"
| summarize sum(DurationMs) by Resource
```
#### Open AI Resource Types
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| summarize count() by ResourceType
```
#### Request Trend
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| summarize ['Duration in Miliseconds'] = make_list(DurationMs) by Resource
```
#### Open AI Activity Timeline
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
|summarize count() by bin(TimeGenerated, {Timespan:grain} ), Resource
```
#### Open AI Deviation of Activities 
```
let baseline = toscalar(AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
|summarize count() by bin(TimeGenerated, 15m)
| summarize avg(count_));
AzureDiagnostics
|summarize count() by bin(TimeGenerated, 15m)
| extend deviation = count_- baseline/ baseline 
| project-away count_
```
#### Open AI Request Response
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where Category == "RequestResponse"
| project OperationName, DurationMs, CallerIPAddress, ResourceGroup, ResourceId
```
#### Creating a completion for the chat message
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where OperationName == "ChatCompletions_Create"
| project TimeGenerated,CallerIPAddress, ResourceId, DurationMs
```
#### New Open AI Deployment
```
AzureActivity
| where OperationNameValue == "MICROSOFT.COGNITIVESERVICES/ACCOUNTS/DEPLOYMENTS/WRITE"
| where ActivitySubstatusValue == "Created"
| extend Role = parse_json(tostring(parse_json(Authorization).evidence)).role
| extend GivenName = parse_json(Claims).name
| project CategoryValue, CallerIpAddress, Caller, GivenName, Role, ActivityStatusValue
```
#### Azure Open AI service failures
```
AzureDiagnostics
| join AzureActivity on ResourceGroup
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where ActivityStatusValue == "Failure"
| extend name_ = tostring(parse_json(Claims).name)
| extend UPN = Caller
| project ActivitySubstatus, ActivitySubstatusValue, HTTPRequest, Resource, OperationName, name_, UPN, CallerIpAddress
```
#### Operations through Request Response
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where Category == "RequestResponse"
| project OperationName, DurationMs, CallerIPAddress, ResourceGroup, Resource
```
#### ChatGPT usage from Logic App API
```
AzureDiagnostics
| join AzureActivity on ResourceGroup
| where Resource contains "GPT3"
| where ResourceProvider == "MICROSOFT.LOGIC"
| where isnotempty(Caller)
| where Caller hassuffix "com"
| extend PlaybookName = resource_workflowName_s
| extend Action = Resource
| distinct Caller, PlaybookName, Action, CallerIpAddress
```
#### Azure OpenAI Content Safety Service: Detect analyzing text or images
```
AzureDiagnostics
| where parse_json(properties_s).apiName == "Content Safety Service"
| where OperationName == "Analyze Image" or OperationName == "Analyze Text"
| distinct CallerIPAddress
```
#### User information using Azure OpenAI Studio
```
AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Azure OpenAI Studio"
| extend parse_json(LocationDetails).city
| extend parse_json(LocationDetails).countryOrRegion
| extend parse_json(LocationDetails).state
| extend parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).latitude
| extend parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).longitude
| extend parse_json(DeviceDetail).browser
| extend parse_json(DeviceDetail).displayName
| extend parse_json(DeviceDetail).operatingSystem
| extend parse_json(DeviceDetail).trustType
| project UserDisplayName, UserPrincipalName, IPAddress, LocationDetails_city, LocationDetails_state, LocationDetails_countryOrRegion, LocationDetails_geoCoordinates_latitude, LocationDetails_geoCoordinates_longitude, DeviceDetail_browser, DeviceDetail_displayName, DeviceDetail_operatingSystem, DeviceDetail_trustType
```
