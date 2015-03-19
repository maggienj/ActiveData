Qb Query Tutorial (incomplete)
=================

Qb queries are very similar to LINQ in C# except the expression tree is written directly as JSON rather than being constructed from native language method calls.   


Simple `from` Clause 
--------------------

All queries must have a `from` clause, which is indicates what data is being queried.  ActiveData has a default documents store, and uses it to translate names to explicit cubes.

```javascript
{"from": "unittest"}

```

In this case, we will get some records from the `unittest` cube.  ActiveData assigns a default limit on all requests to prevent returning overwhelmingly large results by accident.

`limit` Clause
--------------

Use the `limit` clause to get more rows

```javascript
{
	"from": "unittest",
	"limit": 100
}

```

`where` Clause
--------------

Use the `where` clause to restrict our results to those that match

```javascript
{
	"from": "unittest",
	"where":{"eq":{"machine.platform": "linux64"}}
}
```

in this case, we limit ourselves to test results on `linux64` platform.

`select` Clause
---------------

The `unittest` records are quite large, and in most cases you will not be interested in all the properties.  Let's look at how big some test result files can be; list all files over 600 megabytes!  

```javascript
{
	"from":"unittest",
	"select":"run.stats.bytes",
	"where":{"and":[
		{"eq":{"machine.platform":"linux64"}},
		{"gt":{"run.stats.bytes":600000000}}
	]}
}
```

Knowing the size is not enough: What files are they?

```javascript
{
	"from":"unittest",
	"select":[
		"run.stats.bytes",
		"run.files.url"
	],
	"where":{"and":[
		{"eq":{"machine.platform":"linux64"}},
		{"gt":{"run.stats.bytes":600000000}}
	]}
}
```

It appears there are multiple files generated by each test run, at least one of them is the culprit.   Also notice the `select` clause can be an array of properties.

Aggregates
----------

Pulling individual records is unexciting, and it will take forever to get an understanding of the billion+ records behind it.  ActiveData was designed to provide aggregates fast.

`groupby` Clause
----------------

How many of these monster files are there?

```javascript
{
	"from":"unittest",
	"groupby":["machine.platform"],
	"where":{"and":[
		{"eq":{"etl.id":0}},
		{"gt":{"run.stats.bytes":600000000}}		
	]}
}
```

A few notes on this query: First, if there is no `select` clause when using the  `groupby` clause, it is assumed a `count` is requested.  Second, the properties in the `groupby` clause will be included in the result set.  Finally, and most important:

> The `unittest` data cube is a list of **test results** not test runs; each run has multiple results, so if we want to accurately count the number of runs we must pick a specific test result that will act as representative: Your best choice is `etl.id==0`.

How big do these files get?

```javascript
{
	"from":"unittest",
	"select":{"value":"run.stats.bytes","aggregate":"max"},
	"groupby":["machine.platform"],
	"where":{"and":[
		{"eq":{"etl.id":0}},
		{"gt":{"run.stats.bytes":600000000}}
	]}
}
```

At time of this writing we see structured logs of over 1.1 Gigabytes!  No wonder my Python processes were running out of memory! 


Cubes, Data Frames, and Pivot Tables
------------------------------------

The `groupby` clause introduces some problems for data analysis.  The biggest problem is it's unaware of the cardinality of the columns (dimensions) of the data.  Filtering a result-set necessarily impacts the number of rows returned, which does not makes sense for aggregates; one expects filtering to affect the statistics, not the number of statistics.

`edges` Clause
--------------

The `edges` clause works just like `groupby` except it's domain in unaffected by the filter:

```javascript
{
	"from":"unittest",
	"select":{"value":"run.stats.bytes","aggregate":"max"},
	"edges":["machine.platform"],
	"where":{"and":[
		{"eq":{"etl.id":0}},
		{"gt":{"run.stats.bytes":600000000}}
	]}
}
```

**THIS TUTORIAL IS INCOMPLETE**

 
