# Challenge 1

## To bill or not to bill?

> Dear Detective,

> Welcome to the Kusto Detective Agency! We're thrilled to have you on board for an exciting new challenge that awaits us. Get ready to put your detective skills to the test as we dive into a perplexing mystery that has struck Digitown.

> Imagine this: It's a fresh new year, and citizens of Digitown are in an uproar. Their water and electricity bills have inexplicably doubled, despite no changes in their consumption. To make matters worse, the upcoming mayoral election amplifies the urgency to resolve this issue promptly.

> But fear not, for our esteemed detective agency is on the case, and your expertise is vital to crack this mystery wide open. We need your keen eye and meticulous approach to inspect the telemetry data responsible for billing, unravel any hidden errors, and set things right.

> Last year, we successfully served Mayor Gaia Budskott, leaving a lasting impression. Impressed by our work, the city has once again turned to us for assistance, and we cannot afford to disappoint our client.

> The city's billing system utilizes SQL (an interesting choice, to say the least), but fret not, for we have the exported April billing data at your disposal. Additionally, we've secured the SQL query used to calculate the overall tax. Your mission is to work your magic with this data and query, bringing us closer to the truth behind this puzzling situation.

> Detective, we have complete faith in your abilities, and we are confident that you will rise to the occasion. Your commitment and sharp instincts will be instrumental in solving this enigma.

Answer the question - What is the total bills amount due in April?

### Use below query to ingest data into ADX Cluster :

```kusto
.execute database script <|
// The script takes ~20seconds to complete ingesting all the data.
.set-or-replace Costs <| 
    datatable(MeterType:string, Unit:string, Cost:double) [
     'Water', 'Liter', 0.001562, 
     'Electricity', 'kwH', 0.3016]
.create-merge table Consumption (Timestamp:datetime , HouseholdId:string, MeterType:string, Consumed:double)
.ingest async into table Consumption (@'https://kustodetectiveagency.blob.core.windows.net/kda2c1taxbills/log_00000.csv.gz')
.ingest async into table Consumption (@'https://kustodetectiveagency.blob.core.windows.net/kda2c1taxbills/log_00001.csv.gz')
.ingest into table Consumption (@'https://kustodetectiveagency.blob.core.windows.net/kda2c1taxbills/log_00002.csv.gz')
```
## Solution

```
Consumption
| summarize max(Consumed) by HouseholdId, Timestamp, MeterType
| join kind = inner Costs on MeterType
| summarize Sum = sum(max_Consumed*Cost)
```

