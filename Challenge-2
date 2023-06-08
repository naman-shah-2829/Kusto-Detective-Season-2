# Challenge 2

## Catch the Phishermen!

> Hey Detective,

> We've got another case that needs your expertise! The people of our city are being targeted by phishermen, and they need your help to stop them in their tracks.

> The complaints are pouring in, and people are fed up with the sudden increase in phishing calls attempting to steal their identity details. We can't let these scammers get away with it, and we need your help to catch them!

> The police have asked for our assistance, and we've got a massive data set to work with. We've got listings of all the calls that have been made during the week, and we need to find the source of the phishing calls.

> It's not going to be easy, but we know you're up for the challenge! We need you to analyze the data and use your detective skills to find any patterns or clues that could lead us to the source of these calls.

> Once we have that information, the police can take action and put a stop to these scammers once and for all! Are you ready to take on this challenge, detective?

> We've got your back, and we know you can do this! Let's catch those phishermen!

### Use below query to ingest data into ADX Cluster :

```kusto
.execute database script <|
.create-merge table PhoneCalls (Timestamp:datetime, EventType:string, CallConnectionId:string, Properties:dynamic)
.ingest async into table PhoneCalls (@'https://kustodetectiveagency.blob.core.windows.net/kda2c2phonecalls/log_00000.csv.gz')
.ingest async into table PhoneCalls (@'https://kustodetectiveagency.blob.core.windows.net/kda2c2phonecalls/log_00001.csv.gz')
// Last command is running sync, so when it finishes the data is already ingested.
// It can take about 1min to run.
.ingest into table PhoneCalls (@'https://kustodetectiveagency.blob.core.windows.net/kda2c2phonecalls/log_00002.csv.gz')
```

## Solution

```kusto
PhoneCalls
| extend IsHidden = tobool(Properties.IsHidden)
| extend Origin = tostring(Properties.Origin)
| extend Destination = tostring(Properties.Destination)
| where IsHidden == true // hiding the identity
| where EventType == "Connect"
| join kind = inner (
    PhoneCalls
    | where EventType == "Disconnect" // to calculate the call duration
    ) on CallConnectionId
| project Connect_Time = Timestamp, Disconnect_Time = Timestamp1, Origin, Destination, IsHidden, CallConnectionId
| extend Time_Diff_Minutes = datetime_diff('minute', Disconnect_Time, Connect_Time )
| where Time_Diff_Minutes between (0 .. 10) // maximum calls are in this range post this calls are for more than 19 minutes
| extend hour = datetime_part("hour", Connect_Time)
| summarize Call_Count = dcount(Destination) by Origin
| order by Call_Count
```
