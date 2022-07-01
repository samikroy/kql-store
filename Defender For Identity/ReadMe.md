
### Overview
<hr>

#### Log On Summary

```
IdentityLogonEvents
| summarize count() by LogonType
```

#### Log On Status

```
IdentityLogonEvents
|summarize count() by ActionType
```


#### Log On Protocol

```
IdentityLogonEvents
| summarize count() by Protocol
```

#### Query Summary

```
IdentityQueryEvents
| summarize count() by Protocol
```

#### Query Destination Port

```
IdentityQueryEvents
| summarize count() by Protocol
```


#### Top 10 Query Types

```
IdentityQueryEvents
| where isnotempty(QueryType)
| summarize count() by QueryType
| order by count_ desc
| take 10
```

#### Directory Events Protocol

```
IdentityDirectoryEvents
| summarize count() by AccountDomain
```

#### Top 10 Accounts Activity

```
IdentityDirectoryEvents
| where isnotempty(AccountUpn)
| summarize count() by AccountUpn
| order by count_ desc 
| take 10
```

### LogOnEvents
<hr>

#### Resource Access Summary in Devices

```
IdentityLogonEvents
| where LogonType == "Resource access"
| extend FROM_DEVICE_ = tostring(AdditionalFields.["FROM.DEVICE"])
| extend TO_DEVICE_ = tostring(AdditionalFields.["TO.DEVICE"])
| extend Count_ = toint(AdditionalFields.Count)
| summarize sum(Count_) by FROM_DEVICE_, TO_DEVICE_
| order by sum_Count_ desc
```

#### Resource Access Summary in OS

```
IdentityLogonEvents
| where LogonType == "Resource access"
| extend TargetComputerOperatingSystem_ = tostring(AdditionalFields.TargetComputerOperatingSystem)
| extend DestinationComputerOperatingSystem_ = tostring(AdditionalFields.DestinationComputerOperatingSystem)
| extend Count_ = toint(AdditionalFields.Count)
| summarize sum(Count_) by TargetComputerOperatingSystem_, DestinationComputerOperatingSystem_
| order by sum_Count_ desc
```

#### Credentials validation Summary in Devices

```
IdentityLogonEvents
| where LogonType == "Credentials validation"
| extend FROM_DEVICE = tostring(AdditionalFields.["FROM.DEVICE"])
| extend TO_DEVICE = tostring(AdditionalFields.["TO.DEVICE"])
| extend Count = toint(AdditionalFields.Count)
| summarize sum(Count) by FROM_DEVICE, TO_DEVICE
| order by sum_Count desc
```

#### Credentials validation by Domain Controllers

```
let data = IdentityLogonEvents
| where LogonType == "Credentials validation"
| extend TO_DEVICE = tostring(AdditionalFields.["TO.DEVICE"])
| extend Count = toint(AdditionalFields.Count);
data
| summarize sum(Count) by  TO_DEVICE
| order by sum_Count desc
| join kind=inner
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by TO_DEVICE
)
on $left.TO_DEVICE == $right.TO_DEVICE
```

#### Logon Summary by Users

```
let data = IdentityLogonEvents;
data
| extend Count = toint(AdditionalFields.Count)
| summarize LogonSuccess = sumif(Count,ActionType == "LogonSuccess"),LogonFailed= sumif(Count,ActionType == "LogonFailed") by AccountUpn
| join kind = inner
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by AccountUpn
) on $left.AccountUpn == $right.AccountUpn
```

### Query Events
<hr>

#### Device Query Summary
```
let data = IdentityQueryEvents
| extend TO_DEVICE = tostring(AdditionalFields.["TO.DEVICE"]);
data
| summarize count() by QueryType, TO_DEVICE
| join kind =inner 
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by QueryType, TO_DEVICE
) on $left.QueryType == $right.QueryType and $left.TO_DEVICE== $right.TO_DEVICE
```
#### Query Summary from Source Devices

```
let data = IdentityQueryEvents;
data
| summarize count() by DeviceName
| join kind =inner 
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by DeviceName
) on $left.DeviceName == $right.DeviceName
```

### Directort Events
<hr>

#### Timeline on ActionType

```
IdentityDirectoryEvents
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by ActionType 
```

#### Top 10 Actions

```
IdentityDirectoryEvents
| summarize count() by ActionType 
| order by count_ desc
```

#### Account Trend on Directory Events

```
IdentityDirectoryEvents
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by AccountUpn
```


### Alerts
<hr>

#### Alert Trend

```
let data = SecurityAlert
| where ProductName == "Azure Advanced Threat Protection"
| summarize arg_max(TimeGenerated,*) by SystemAlertId;
data
| summarize count() by AlertName
| join kind=inner 
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by AlertName
) on AlertName
```

#### Impacted Hosted

```
let data = SecurityAlert
| where ProductName == "Azure Advanced Threat Protection"
| summarize arg_max(TimeGenerated,*) by SystemAlertId
| mv-expand todynamic(Entities)
| where Entities["Type"] == "host"
| where isnotempty(Entities["HostName"])
| extend HostName = tostring(Entities["HostName"]);
data
| summarize count() by HostName
| join kind=inner 
(
data
| make-series Trend = count() default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by HostName
) on $left.HostName == $right.HostName
```
