
## Overview

#### Log On Summary
<hr>

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

