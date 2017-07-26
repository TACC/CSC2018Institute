# Analyzing Large CSVs with Dask Distributed

While this is not a course in Big Data, we do want to emphasis that clouds provide users with computing capabilities that exceed well beyond their own personal computers. In fact, industry giants such as Google and Amazon do staggering amounts of computation "in the cloud".

## Reading CSV files

To get started, we need to read our raw data into a dask dataframe. Once we have imported our packages and created our client, dask makes this as easy as pandas does.

```
csv = '/root/stackoverflow.csv'
df = dd.read_csv(csv, parse_dates=['creationdate', 'acceptedAnswerCreationDate'])
df = client.persist(df)
progress(df)
```

A couple of notes: 
  * We read the csv directly in using the `read_csv` method. This mirrors the pandas API.
  * We call the `client.persist` method. The dask client tells the scheduler to cut up the data into dozens of smaller pandas dataframes scattered across the cluster. 
  * The `progress` function graphs the progress of distributing the work to the cluster. Note how fast it completes.
  

## Basics Methods

Let's see how big the dataset is that we are working with:

```
len(df)
14015379
```
With over 14 million rows, we see it takes a few seconds for the result to come back. 


Familiar pandas methods are available for basic data exploration. Let's look at what column are present using the columns attribute.

```
Exercise. 
  * What is the most popular day for asking a question> What is the least popular day?
  * Does the number of answers a question has seem to depend on the day it was asked? Compute the average answer count by day of the week.
  * Break down the questions by total number of answers they receive. What are the top 10 most likely answer counts
for a question?
```

First, we investigate the CreationDayOfWeek column. To make sure it's the data we think it is, let's use the unique() method to get the unique values:
```
df.CreationDayOfWeek.unique().compute()
```

> One difference between dask and pandas is we have to call compute() to tell dask to schedule the computation.

For the first question, we can look at a related method, the `value_counts()` method.

For part 2, we need the `groupby` method. We call `df.groupby(df.creationDayOfWeek)` to group the elements by the day of the week of the posting. Then, we chain to that `.answercount.mean()` since the average number of answers is what we are interested in. Don't forget the `.compute()` at the end!

For the last part, we can use the `value_count` again. If we just want the 10 most likely answercounts, we can call the `.head(10)` method.


## Null Values
Just as with normal pandas, we can make use of `isnull()` and `notnull()`
```
Exercise. What percentage of questions have an accepted answer?
```
We can do this using the accepteranswerid field.


## Working with Dates
For questions that had an accepted answer, let's explore how long it took before the accepted answer was posted.

With dask, just as with normal pandas dates, we can subtract the values as long as pandas recognizes they are dates. The subtraction will result in a timedelta object.

To get started we should create a new dataframe with only the posts with an accepted answer. Then we can:
  * Filter out rows for which the creation date of the accepted answer was less than the creation date of the post. (How could this happen?) because this will throw off our calculations.
  * Create a new column containing the difference between the dates.
  * Create another new column using the `np.timedelta64(1, 'D')).astype(int)` to convert the timedelta to an integer by dividing.
  * Plot the result of `count()` after grouping by the number of days (resp, weeks, months). 

