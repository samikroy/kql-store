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
| render columnchart 
```
