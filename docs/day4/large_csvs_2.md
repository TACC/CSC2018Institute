# Analyzing Large CSVs Part 2

In this section, we turn to the 2015 Yellow Cab Data Set. The kinds of operations we use are similar to those from the previous section. (Special thanks to Matt Rocklin, founder of dask and many other great open source Python libraries, for the permission to reuse the materials below).

> Remember that all CSV files need to be downloaded to all nodes.

Let's start by reading in the dataset. We can use the `read_csv()` function and pass it all the files using a glob. We also need to have pandas parse the date fields.

```
csv = '/root/yellow_tripdata_2015-*'
df = dd.read_csv(csv, parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'])
```

Do a similar basic exploration of the data. You could begin with these questions:
  * How many total cab trips are in the dataset?
  * How do the trip distances vary as a function of the number of passengers?
  * Compute average tip, as a fraction of the fare amount, and explore how that fraction varies as a function of i) the hour of the day and ii) the day of the week.


The payment type is an integer which codes for the following payment types:
```
1: 'Credit Card'
2: 'Cash', 
3: 'No Charge', 
4: 'Dispute', 
5: 'Unknown', 
6: 'Voided trip'
```
You could investigate:
  * How do tip amounts vary by payment type?
  * How do tips on cash compare to tips on credit cards?
  
Finally, let's explore the value of computing an index. Let's find an efficient solution to getting the first 10 cab rides of every month.

To set an index on column, we use
```
df.set_index('<column_name')
```
however we must the persist that to cluster by sending it to `client.persist()`. 

One approach to the above:
  * Set an index on the pickup time date time
  * for each month, call head on the first day of the month
Look at ways to efficiently plot these data.

