### Microsoft Coplit for Security Monitoring Workbook Queries

#### Use the below to deploy the workbook in your environment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FCopilotForSecurity%2FCopilotforSecurityMonitoringARM.json)

### Use below queries for know the specific details about the Microsoft Copilot for Security


### Overview 

#### User Signins across the globe

```
union (SigninLogs
| where AppDisplayName == "Medeina Service"
| extend City = tostring(parse_json(Location).city)
| extend ST = parse_json(Location).state
| extend LAT = tostring(parse_json(tostring(parse_json(Location).geoCoordinates)).latitude)
| extend LONG = tostring(parse_json(tostring(parse_json(Location).geoCoordinates)).longitude)),
(AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Medeina Service"
| extend City = tostring(parse_json(LocationDetails).city)
| extend ST = parse_json(LocationDetails).state
| extend LAT = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).latitude)
| extend LONG = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).longitude))
| extend City = "Mumbai", LAT = "19.076000213623048", LONG = "72.87770080566406"
| summarize count() by  City, LAT, LONG

```

#### Process time with each apps

```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where ResourceDisplayName has "Medeina"
| project TimeGenerated, ResourceDisplayName, AppDisplayName, toint(ProcessingTimeInMs)
| summarize Total_Processing_Time_in_ms = sum(ProcessingTimeInMs) by AppDisplayName
```

#### Failed MFA authentication against the Standalone experience.

```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Medeina Portal" 
| where ResultType == "50074" 
| summarize count() by UserPrincipalName
| extend Status = 3
```

#### Average of SCU processing time

```
AADNonInteractiveUserSignInLogs
| where ResourceDisplayName has "Medeina"
| project TimeGenerated, ResourceDisplayName, AppDisplayName, toint(ProcessingTimeInMs)
| summarize Avg_Processing_Time_in_ms = avg(ProcessingTimeInMs) by ResourceDisplayName
```

### Signins

#### Successful Login Over Time
```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where AppDisplayName ==  "Medeina Portal" 
| where ResultType ==0
| summarize count() by bin(TimeGenerated, {Timespan:grain})
```

#### Top 5 Users with Successful Login

```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where AppDisplayName ==  "Medeina Portal" 
| where ResultType == 0
| summarize count() by UserPrincipalName
| order by count_ desc
| limit 5
```

#### Successful Logins

```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where ResourceDisplayName == "Medeina Service"
| where ResultType ==0
```

#### Non Interactive User SignIns Over Time

```
AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Medeina Service"
| summarize count() by bin (TimeGenerated,{Timespan:grain})
```


#### Top 5 Users Non Interactive SignIns

```
AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Medeina Service"
| summarize count() by UserPrincipalName
| order by count_ desc
| limit 5
```

#### Non Interactive User SignIns from Locations

```
AADNonInteractiveUserSignInLogs
| where AppDisplayName == "Medeina Service"
| extend Burg = parse_json(LocationDetails).city
| extend ST = parse_json(LocationDetails).state
| extend LAT = parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).latitude
| extend LONG = parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).longitude
| project TimeGenerated, Identity, Burg, ST, LAT, LONG
| order by TimeGenerated desc
```


#### Failed MFA authentication against the Standalone experience

```
SigninLogs 
| where AppDisplayName == "Medeina Portal" 
| where ResultType == "50074" 
| extend city = LocationDetails.city 
| extend state = LocationDetails.state 
| extend region = LocationDetails.countryOrRegion 
| extend latitude = parse_json(tostring(LocationDetails.geoCoordinates)).latitude 
| extend longitude = parse_json(tostring(LocationDetails.geoCoordinates)).longitude 
| project UserDisplayName, UserPrincipalName, UserType, city, state, region, latitude, longitude, AADTenantId
```

#### Failed Logins against the Standalone experience

```
SigninLogs 
| where AppDisplayName == "Medeina Portal" 
| where ResultType != 0 
| extend city = LocationDetails.city 
| extend state = LocationDetails.state 
| extend region = LocationDetails.countryOrRegion 
| extend latitude = parse_json(tostring(LocationDetails.geoCoordinates)).latitude 
| extend longitude = parse_json(tostring(LocationDetails.geoCoordinates)).longitude 
| project UserPrincipalName,IPAddress, UserType, city, state, region, latitude, longitude, AADTenantId
```

#### Success vs Failed Logins for Users

```
union SigninLogs,
AADNonInteractiveUserSignInLogs
| where TimeGenerated >ago(30d)
| where AppDisplayName ==  "Medeina Portal" 
| summarize SuccessfulLogin = countif(ResultType == 0), FailedLogin = countif(ResultType != 0) by UserPrincipalName
```

### Activity

#### Average of SCU processing time by resource

```
AADNonInteractiveUserSignInLogs
| where ResourceDisplayName has "Medeina"
| summarize Avg_Processing_Time_in_ms = avg(toint(ProcessingTimeInMs)) by  AppDisplayName
```

#### Total of SCU processing time by resource

```
AADNonInteractiveUserSignInLogs
| where ResourceDisplayName has "Medeina"
| summarize Total_Processing_Time_in_ms = sum(toint(ProcessingTimeInMs)) by  AppDisplayName
```

#### Users created or changed Copilot for Security SCUs

```
AzureActivity
| where OperationNameValue == "MICROSOFT.SECURITYCOPILOT/CAPACITIES/WRITE"
| where ActivityStatusValue == "Success"
| summarize count() by Caller
```

#### Role that enabled  Copilot for Security

```
//Role that enabled  Copilot for Security
AzureActivity
| where OperationNameValue == "MICROSOFT.SECURITYCOPILOT/REGISTER/ACTION"
| where ActivityStatusValue == "Success"
| extend role_ = tostring(parse_json(tostring(parse_json(Authorization).evidence)).role)
| summarize count() by role_
```

#### Capacity changes captured in CloudAppEvents

```
CloudAppEvents
| where ActionType == "Write Capacities"
| project AccountDisplayName, City, CountryCode, ISP, ObjectName
```


#### Identify enabling Copilot for Security in the CloudAppEvents log

```
CloudAppEvents
| where ActionType == "Register Microsoft.SecurityCopilot"
| project AccountDisplayName, City, CountryCode, ISP, ObjectName
```

#### Activities from Unified Console

```
AADNonInteractiveUserSignInLogs 
| where AppDisplayName == "Microsoft 365 Security and Compliance Center" 
| where ResourceDisplayName has "Medeina" 
| extend city_ = tostring(parse_json(LocationDetails).city) 
| extend countryOrRegion_ = tostring(parse_json(LocationDetails).countryOrRegion) 
| extend latitude_ = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).latitude) 
| extend longitude_ = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).longitude) 
| extend state_ = tostring(parse_json(LocationDetails).state) 
| project TimeGenerated, Identity, UserType, UserPrincipalName, city_, countryOrRegion_, state_, latitude_, longitude_, ResourceDisplayName, AppDisplayName, ProcessingTimeInMs
```

#### Identifies when the Intune extension for CfS has been used

```
//Identifies when the Intune extension for CfS has been used

AADNonInteractiveUserSignInLogs 
| where AppDisplayName == "Microsoft Intune portal extension" 
| extend city_ = tostring(parse_json(LocationDetails).city) 
| extend countryOrRegion_ = tostring(parse_json(LocationDetails).countryOrRegion) 
| extend latitude_ = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).latitude) 
| extend longitude_ = tostring(parse_json(tostring(parse_json(LocationDetails).geoCoordinates)).longitude) 
| extend state_ = tostring(parse_json(LocationDetails).state) 
| project TimeGenerated, Identity, UserType, UserPrincipalName, city_, countryOrRegion_, state_, latitude_, longitude_, ResourceDisplayName, AppDisplayName, ProcessingTimeInMs;
```

#### Current Copilot Instances from Azure Resource Graph Explorer

```
resources
| where type == "microsoft.securitycopilot/capacities"
| extend GEO = tostring(properties['geo'])
| extend NumberofSCUs = tostring(properties['numberOfUnits'])
| extend State = tostring(properties['provisioningState'])
| project ComputeName = name, location, resourceGroup, ['type'], NumberofSCUs, GEO, State, subscriptionId, tenantId
```

