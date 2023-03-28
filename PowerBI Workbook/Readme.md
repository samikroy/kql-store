

```

PowerBIActivity
| summarize count() by Activity, bin(TimeGenerated,{TimeRange:grain})

```

