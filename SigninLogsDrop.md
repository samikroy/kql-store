
## Signin LOgs not ingesting in Sentinel 

Recently a sudden drop is observed for Sign in Logs in Azure Monitor & Microsoft Sentinel.

Seems Azure Active Directory SignInLogs logs are lagging in ingestion to Sentinel.

Hence, formulated this query to identify the gap


```
SigninLogs
| where TimeGenerated > ago(2d)
| summarize LoginCount = count() by bin(TimeGenerated,1h)
| order by TimeGenerated desc
| render timechart
```


![SigninLogsDrop](https://user-images.githubusercontent.com/20562985/175232795-6152dee1-38f9-471e-b99b-aec4814177d1.png)

