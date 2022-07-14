## Non Interective SingnIn Logs

### Top 10 Applications having failed logins

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

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FNoiseInNonInteractiveSignins
%2Fazuredeploy.json)
