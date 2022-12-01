

Query 1 : Take 1 from SignIn Logs to verify the data set existence.

```
SigninLogs
| take 1
```

Query 2 : Login summary by UPN

```
SigninLogs
| summarize count() by UserPrincipalName
```
Query 3 : Login Summary by Location over time.

```
SigninLogs
| summarize count() by bin(TimeGenerated, 1h),Location
| render timechart 
```
Query 4: Analyze the Effectiveness Conditional Access Policies over time
```
SigninLogs
//| take 1
| mv-expand ConditionalAccessPolicies
| extend CA_DisplayName= tostring(ConditionalAccessPolicies.displayName)
| extend CA_Result= tostring(ConditionalAccessPolicies.result)//, UserPrincipalName, TimeGenerated
| summarize count() by CA_DisplayName, CA_Result, bin(TimeGenerated,1h)
| order by count_ asc 
| render columnchart  
```
