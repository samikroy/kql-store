Data Ingestion Trend

```
Syslog
| where SeverityLevel in~ ({SeverityLevel}) or '*' in~ ({SeverityLevel})
| summarize count() by HostName, bin(TimeGenerated,{TimeRange:grain})
```

Heartbeat

```
Syslog
| summarize arg_max(TimeGenerated,*) by HostName
| extend ['Last Log Seen Ago'] = datetime_diff('second',now(), TimeGenerated)
| order by ['Last Log Seen Ago'] desc 
| project HostName, ['Last Log Seen Ago']
| join (Syslog
        | make-series SyslogIngestionTrend = count(SeverityLevel) default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by HostName) on HostName
```


### HostNames

```
Syslog
| summarize count() by HostName
```

### Facility

```
Syslog
| summarize count() by Facility
```

### Severity Level

```
Syslog
| summarize count() by SeverityLevel
```

### Severity Trend Summary

```
Syslog
| summarize count(SeverityLevel) by SeverityLevel 
| extend jkey = 1
| join (Syslog
| make-series Trend = count(SeverityLevel) default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by SeverityLevel) on SeverityLevel
```

### Syslog Trend

```
Syslog
| summarize SyslogEventCount=count(SeverityLevel) by Facility, HostName
| join (Syslog
        | make-series SyslogTimeLine = count(SeverityLevel) default = 0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Facility,HostName) on Facility,HostName
| project-away Facility1, TimeGenerated
```

### Timeline
```
Syslog
| extend Pack=pack_all()
| extend TimeFromNow = now() - TimeGenerated
| extend TimeAgo = strcat(case(TimeFromNow < 2m, strcat(toint(TimeFromNow / 1s), ' seconds'), TimeFromNow < 2h, strcat(toint(TimeFromNow / 1m), ' minutes'), TimeFromNow < 2d, strcat(toint(TimeFromNow / 1h), ' hours'), strcat(toint(TimeFromNow / 1d), ' days')), ' ago') 
| project ["Time"]=strcat('🕒', TimeAgo), HostName, SeverityLevel, Facility, SyslogMessage, ProcessName, ["Details"]=Pack

```
