## Non Interective SingnIn Logs

### Here is the query to test the same 

```
AADNonInteractiveUserSignInLogs
| extend ErrorCode = tostring(parse_json(Status).errorCode)
| summarize TotalLogins = count(),
SuccessfulLogins = countif(ErrorCode == 0),
FailedLogins = countif(ErrorCode != 0),
Users = make_set(UserPrincipalName)
by AppDisplayName, ClientAppUsed
| where FailedLogins > 0
| project UserCount = array_length(Users),AppDisplayName, ClientAppUsed
| take 10
| render columnchart 
```


### Top 10 Users having failed logins

``` 

AADNonInteractiveUserSignInLogs
| extend ErrorCode = tostring(parse_json(Status).errorCode)
| summarize TotalLogins = count(),
SuccessfulLogins = countif(ErrorCode == 0),
FailedLogins = countif(ErrorCode != 0),
Apps = make_set(AppDisplayName),
ClientApps = make_set(ClientAppUsed)
by UserPrincipalName
| where FailedLogins > 0
| order by FailedLogins desc
| project-away TotalLogins
| take 10
| render columnchart 
