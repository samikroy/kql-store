## Deviation in Security Events

Generally we deploy OMS/ Log Analytics in Domain Controllers to capture Security Events.
Now every machine is has its own traffic of events. But until they carry out the same responsibility like Domain Controllers the average events should be similar.

Here is a query to understand the deviation of a machine from the average.


Query

```

// Defining lookback Time.
let lookbacktime = 1h;
//Deriving the average.
let BL = toscalar(SecurityEvent
| where TimeGenerated > ago(lookbacktime)
| summarize count() by Computer
| summarize avg(count_));
//Fetching actual event count
SecurityEvent
| where TimeGenerated > ago(lookbacktime)
| summarize count() by Computer
// Calculating the Deviation
| extend Deviation = (count_ - BL) / BL
| project-away count_
// rendering visual
| render columnchart 


```

Visual

![image](https://user-images.githubusercontent.com/20562985/175227526-724ca50a-1984-4cd9-a070-f8f006b7415e.png)
