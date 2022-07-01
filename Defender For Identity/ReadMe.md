
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
