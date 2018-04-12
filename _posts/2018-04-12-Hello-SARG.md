--- 
layout: post
title: "Hello SARG !"
author: "Author"
comments: true
tags : [SQL, RDBMS, SARGability]
---

No, not your Beetle Bailey Sarge. 

This story begins with LINQ and composite keys in EF. (<small>in fact, many programmer stories usually begin with LINQ. It’s a veritable treasure trove of stories</small>). Here we have composite keys and query plans and cryptic acronyms all playing ball really well together, only in a fashion nobody understands. 

What I needed was the LINQ equivalent of a SQL IN query. Consider a table with a single column primary key called Id. The query to get records with a IN clause via LINQ is - 

```
    from entity in db.Table where computedList.Contains(entity.Id) select x.
```

Therefore, extrapolating merrily, you reason that tables with Composite keys would need a join as you can't match multiple values in the contains option. And so you write

```
    from entity in db.Table 
    join pair in locals on new { entity.Id1, entity.Id2 }
                        equals new { Id1 = pair.Item1, Id2 = pair.Item2 }
    select entity
```
	
Boom! Welcome to error message 101. (<small> ps. Did you ever hear that joke about the guy who wanted to write great things, move people to joy and tears through his words and ended up writing error messages for Microsoft</small>). Here is an example in action, the result of executing the above LINQ -

	...Only primitive types or enumeration types are supported in this context.

What it means here is EF and SQL Server have no idea how to join a table in database with a list in memory. It cannot be done. You can either have a SQL query (generated through the IQueryable implementation of LINQ) or you fetch all database data into memory and then do the join. The latter is a very bad idea in a table with even 200000 rows, let alone millions. 

So how do you write the LINQ such that you get SQL which runs in SQL server? Turns out that this problem is quite intractable, with no elegant solutions. 

You have one SQL equivalent which uses LINQ's 'Contains' option awkwardly - 

```
    from entity in db.Table
    where computed.Contains("Id1=" + entity.Id1 + "," + "Id2=" + entity.Id2)
    select entity
```

In this case, you have created your list 'computed' with key values which are a concatenation of all key columns as strings. In your LINQ query, you create equivalent keys from all of the database rows and compare this generated key with your predefined list. This generates a SQL of the sort

```
Select * from Table Where ("Id1=" + Id1 + "," + "Id2=" + Id2) in ( "Id1=Id1,Id2=Id2", "Id1=Id2221,Id2=Id2222")
```

<small>*Note:* Not exact, but that’s handwritten un-compiled SQL, so there may be issues, but you get the drift.</small> 

This is messy. But that is not the main problem. Quoting from Stack Overflow - "a more important problem is that this solution is not **sargable**, which means: it bypasses any database indexes on Id1 and Id2 that could have been used otherwise. This will perform very very poorly."

Note: For an in-depth discussion of Composite Keys and EF, check out this [Stack Overflow question](https://stackoverflow.com/questions/26198860/entityframework-contains-query-of-composite-key/26199792) or this [link]( https://www.codesd.com/item/entityframework-contains-a-composite-key-query.html).

## Enter the SARG !
 
I had encountered this SARG before, but had forgotten the details of it, so had to look it up. Turns out SARGability is a big deal in SQL server and other RDBMSs. 

So what is **SARG**? 
It’s an acronym word derived from Search and Argument. An alternative usage is **SARGable**, which means _'Search Argument Able'_. As per Wikipedia **SARGable** is defined as _'In relational databases, a condition (or predicate) in a query is said to be sargable if the DBMS engine can take advantage of an index to speed up the execution of the query'_.

A technical definition would be _'Sargability is the query's ability to traverse the b-tree index using the binary search method that relies on half-set elimination for the sorted items array_. In SQL, it would be displayed on the execution plan as a "index seek".'

Now why is this important? 

Because indexes are key to fast and efficient RDBMS queries and stored procedures. <small> *Note:* At this point, if you are not interested in fast and efficient database queries, you can now go read something else. For the rest however, hopefully this will help.</small>

Databases have **Index**es. In fact so important and ubiquitous are database indexes that RDBMSs create all manner of indexes automatically for you. Primary Keys, Foreign Keys, Unique contraints - all are indexes created silently behind the scenes in any respectable database. However, indexes are no good if you query cannot take advantage of them. That is where SARGability comes in. 

_*IMPORTANT NOTE*_ - A non sargable query will not use your indexes. 

Instead it will fall back to  index scans/table scans (indexed and non-indexed tables respectively) instead of index seeks, and it will do so all of the time, destroying the entire purpose of indexes and our goal of fast efficient queries. 

To understand why this happens, We will look a simple but oft-repeated SQL need. Consider a Table called Customer with an primary key fields - **Id**, and a **LastName** varchar column with an index on it. We need to retrieve Customers whose last name is Smith. So let us look at two simple queries which can do this.
 
```
1. Select * from Customer where LastName = 'Smith'
```
```
2. Select * from Customer where Upper(LastName) = 'SMITH'
```

A quick look at the query plans for each 

#### Option 1
![Option 1](/blog/img/posts/sarg-query-plan-seek.png). 

#### Option 2
![Option 1](/blog/img/posts/sarg-query-plan-scan.png). 

Did you see what happened? The query plan changed from _index **seek**_ to _index **scan**_. 

In this case my table had two columns, with indexes defined on each. A primary key and a non-primary index on LastName and still we have a scan. 

In real world scenarios, with multiple tables, joins and complex clauses and functions, a predicate (where clause) or join condition can easily go from a index seek to an index scan to a table scan. SARG-ability greatly determines the choice of indexing _used_, **OR** _not used_, and thus has a direct impact on the execution time of queries.

## Breaking SARGability and how to fix.

Many scenarios break Sargability. General thumbrules of breaking Sargability and things not to do include -

### Implicit/explicit datatype conversion before comparision

Given that column B is defined as a varchar.

```
Select * from F WHERE B = 1 
``` 
is not sargable. Do the following instead. 

```
Select * from F WHERE B = '1' 
```
### Applying functions/operations to indexed columns before comparisions or joins. 

In the following case, 
```
Select * from F where DateDiff(day, ODate, GetDate()) > 7
``` 
is bad. The SQL optimizer can't use an index on the field, even if one exists. It will literally have to evaluate this function for every row of the table. The better option is to have only the ODate column in where clause. A rewrite would look like this - 
```
Select * from F where ODate >= DateAdd(day, -7, Cast(GetDate() as Date))
```
Basically, If possible, use functions in reverse. Instead of ```WHERE FUNCTION(Field) = 'BLAH'```, do ```WHERE Field = INVERSE_FUNCTION('BLAH')```.

Other examples - 
```
Bad: Select ... WHERE isNull(FullName,'Ed Jones') = 'Ed Jones'
Fixed: Select ... WHERE ((FullName = 'Ed Jones') OR (FullName IS NULL)) (Note that this is bad for other reasons - not being able to cache a query plan).

Bad: Select ... WHERE SUBSTRING(DealerName,4) = 'Ford'
Fixed: Select ... WHERE DealerName Like 'Ford%'

Bad: SELECT ...WHERE Number + 0 = 42
Fixed: SELECT ...WHERE Number  = 42 - 0
```
<small>Note: Read somewhere that An explicit CAST of a DATE column to DATETIME still leaves the predicate SARGable. This is an exception that’s been specifically coded into the optimiser. Not verified though.</small>

### Wild-cards

Leading on from the last query in the list just above, do you see the wildcard in 
```Select ... WHERE DealerName Like 'Ford%'``` . Note that the wildcard is not a leading wildcard.

Leading wildcards are bad. This is bad.

```Select ... WHERE DealerName Like '%ord%'```

This basically turns the query into a full-text search query with most probably a table scan, or in slightly better case, an index scan.

#### Tip : Indexes on computed columns.

Sometimes the use of functions and so on is unavoidable. The general solution is to add a calculated column for the field value returned by the function and then create a non-clustered index for thet calculated column.

Note that this computed column is not required to be persisted.

Consider the query 
```
SELECT LastName  FROM Customers where LEFT(LastName,1)='K'
```

In this case, add a computed column _'LastNameStartsWith'_ to hold ```Left(LastName, 1)```, and add a non-clustered index on _'LastNameStartWith'_. **_Be aware that this is a double edged sword_** however, as you may end up creating lots of indexes for every use combination. Each index addition creates more runtime maintenance work for the database and ultimately may be counter effective. Be discreet.

### References 

- [Sargable date comparisions](https://stackoverflow.com/questions/10853774/is-this-date-comparison-condition-sarg-able-in-sql)

- [Composite Keys and Contains](https://stackoverflow.com/questions/26198860/entityframework-contains-query-of-composite-key/42412401)

- [SQl Sargability](https://stackoverflow.com/questions/799584/what-makes-a-sql-statement-sargable)

- [Sargable where clauses](https://dba.stackexchange.com/questions/132437/sargable-where-clause-for-two-date-columns)

- [Inverting Joins](https://tsqlninja.wordpress.com/2012/05/29/join-me-inverting-joins-to-maintain-sargability/)

- [Sql Server Sargable functions](http://blogs.lobsterpot.com.au/2010/01/22/sargable-functions-in-sql-server/)