SQL References

The following text will only serve as a reference and will only give a brief overview of SQL syntaxes and its capabilities. It goes over the most commonly used statements and the most common mistakes. The examples are designed to be brief but contain as much different statements in certain cases to illustrate the capabilities of SQL. The information contained in here will not be complete as the text only serves as a reference.

Table Of Contents:

[TOC]

### Basic Select Statements

```sql
select *
from [server_name].[database_name].[table_name] WITH (NOLOCK)
```

select * selects all the columns in the table

> WITH (NOLOCK) makes the table alterable when a query is running on the table. The downside will be that new data may come in and the query will reflect the older data.

```sql
select distinct
	Name
	,Gender
from [server_name].[database_name].[table_name]
```

select distinct will return the unique values of the selected combination. In the example above, it will return unique combinations of **both** the columns Name and Gender.

```sql
select top 100
	Name
	,Gender
from [server_name].[database_name].[table_name]
--LIMIT 100
```

A select statement can also just return the top n results as illustrated by the example above. Sometimes the top (n) statement is not available in some SQL languages (i.e MySQL), and a **LIMIT** statement at the end of the select statement as illustrated by the **LIMIT** statement in the example above can be used instead and will return the same exact result.

### Comments

```sql
select
	*
from [server_name].[database_name].[table_name]
--this is a comment
/* this is a multi
line comment */
```

Comments are specified with a double dash **--** for a single line comment or enclosed in **/*** and ***/** for a multi line comment.

### Selecting multiple columns and column aliases

```sql
select
	ClaimID
	,MemberID
	,LineID
	,CID = ClaimID
	,ClaimID as CID
	,DateOfBirth DOB
from [server_name].[database_name].[table_name]
```

You can select certain columns by specifying the actual column names. Column aliases can be given by specifying the alias followed by the **=** sign or by using **as** and the alias after the column name like the example above or by just specifying the alias after the column.

### Table Aliases

```sql
select
	a.Name
	,a.Gender
	,a.MemberID
	,b.ProviderName
from [server_name].[database_name].[table_name] as a
join [server_name].[database_name].[table_name] b
on
	a.MemberID = b.MemberID
```

Tables can be aliased by specifying an alias at the end of the table name as illustrated above by the names **a** and **b**. The keyword **as** can be omitted when aliasing a table (just like when aliasing a column). After a table is aliased any columns within that table can be selected using the following format **table_alias.column_name** (i.e. a.Name in the example above). When querying just one table, aliases are usually not used and not necessary. However, join statements usually require table aliases to avoid writing out the whole table name when calling out columns that have the same name in multiple different tables or when joining key columns.

> If table **a** and **b** both contain the column **Name**, the following SQL statement will result in an *error*:
>
> ```sql
> select
> 	Name
> from [server_name].[database_name].[table_name] a
> join [server_name].[database_name].[table_name] b
> on
> 	a.MemberID = b.MemberID
> ```
>
> Using either **a.Name** or **b.Name** will resolve the error and will return the respective columns from the respective tables.

Join statements will be discussed in another section below.

### Filters (Where Statements)

```sql
select
	*
from [server_name].[database_name].[table_name]
where
	(ClaimID = '1012394123'
	and MemberID = '912C8123')
	or IncurredDate = '2019-01-31'
	or 1 = (case when Gender = 'Male' then 1 
            	 when Gender = 'Female' then 1
            	 else 0 end)
    or PaidDate >= '2019-01-01'
    or PaidDenied <> 'D'
    or (MemberName like 'Johnny%'
    and MemberName not like '%Bravo'
```

Tables can be filtered using the **where** statement. Logical operators such as **and** and **or** can be used to further specify the filter.

> A case when statement can also be used as a filter. Usually a 1 = 1 expression is used to validate the where statement. Multiple levels of case when statements can be used to further  break down the where statements at a more granular level.

> **Like** statements can be used to match words with the wildcard operator **%**. For example, like **'apple%'** will return all sentences that start with **'apple'**, likewise **'%apple'** returns all sentences that ends with **'apple'** and **'%apple%'** return all sentences that contain the word **'apple'**.

### Case When Statements

```sql
select
	case when Gender = 'Male' then 1
		 when Gender = 'Female' then 2
		 else 0 end as GenderIdentifier
	,case when State in ('CA', 'WA') then 'West Coast'
		  when State in ('NY','CH') then 'East Coast'
		  else 'Mid Coast' end as Coast
	,case when Age <= 5 then 'Baby'
		  when Age <= 15 then 'Teen'
		  when Age <= 20 then 'Young Adult'
		  when Age between 21 and 29 then 
		  		case when Gender not in ('Male','Female') then 'xyz Adult'
		  			 else 'Normal Adult'
		  when Age <= 50 then 'Older Adult'
		  else 'Oldest Adult' end as adult_class
from [server_name].[database_name].[table_name]
```

Case when statements are usually used to recategorize the datasets depending on the level of granularity desired. Multiple case when statements can be **nested** as shown by the example above. Case when statements are also **evaluated from the top to bottom**, meaning if there are multiple conditions that satisfy the criteria, the data will be categorized to the first category in which the first condition satisfies. Example if the member age is 3 then the class will be 'Baby' instead of all the other available classes since it is evaluated first even though it satisfies all other classes' criteria.

> **in** statements can be used to evaluate lists. It checks whether or not the row is an element of the list provided.
>
> **not** statements usage are the same as the not logical operator (**!**)

### Summary: Logical Operators

The followings lists all the logical functions that are commonly used for querying:

|  Logical Operator   |                            Usage                             |
| :-----------------: | :----------------------------------------------------------: |
|          !          |             not operator, changes false to true              |
|          =          |                        equal operator                        |
|      <> or !=       |                      not equal operator                      |
|    <; <=; >; >=     | less than; less than/equal to; more than; more than/equal to |
| between ... and ... |                  between 2 specified ranges                  |
|        like         |         commonly used with **%** (wildcard operator)         |
|         in          |       check if the data is contained in the given list       |

> The in statement are also commonly used alongside subqueries, take a look at the following example:
>
> ```sql
> select
> 	*
> from [server_name].[database_name].[table_name]
> where
> 	Names in (select distinct
>           			Names
>               from [server_name].[database_name].[table_name2]
>               where Gender = 'Male')
> ```
>
> The query above will return a table of names in table 1 that has the names in table 2 which Gender is Male. A distinct can be used in defining the subquery, although, it is *unnecessary*.

### Group By

Group by statements are used when using aggregate functions (sum, avg, count, count(distinct )) and less commonly to return a unique combination of columns (which has the same functionality as a select distinct statement).

```sql
select
	Name
	,MemberID
	,ClaimID
	,case when Age <= 10 then 'Baby'
	  	  when Age <= 20 then 'Young Adult'
	  	  when Age <= 30 then 'Adult'
	  	  else 'Senior' end as AdultClass
	,count(distinct ID) as ct_distinct_id
	,count(ID) as ct_id
	,sum(numbers) as sum_num
from [server_name].[database_name].[table_name]
where
	Gender <> 'Male'
group by
	Name
	,MemberID
	,ClaimID
	,case when Age <= 10 then 'Baby'
	  	  when Age <= 20 then 'Young Adult'
	  	  when Age <= 30 then 'Adult'
	  	  else 'Senior' end
```

All the non-aggregated columns are included under the group by statement. As in the example above, since the 4 columns: Name, MemberID, ClaimID and AdultClass are not included in any of the aggregation functions, they are included under the group by statements. The group by order does not matter as long as all non aggregated columns are included (i.e. the group by statement above can be rewritten as: ClaimID, AdultClass, MemberID, Name or any variations thereof).

However, in certain SQL versions, **aliases cannot be used** when listing the columns under the group by statement as illustrated by the example above where the **AdultClass** aliased column is not simply listed under the group by statement but the whole case when statement have to be **copied over**.

> The group by statement above can also be replaced with the following statement: 
>
> group by 1, 2, 3, 4
>
> Where the numbers 1, 2, 3 and 4 refer to the column numbers/location on the select statement.
>
> This approach, however, is not recommended as it discourages readability but is often used when "scripting"

### Having

The Having statement behaves similarly to the where statement, the only difference is that the Having statement filters out conditions after the table is being grouped:

```SQL
select
	Date
	,sum(Visits) as TotalVisits
from [server_name].[database_name].[table_name]
group by
	Date
Having sum(Visits) >= 100
```

The query above will only return dates in which the TotalVisits are greater than or equal to 100. The number of visits on each day is first grouped and then finally filtered out by the Having statement. Just like the group by statement, aliases often times do not work with the Having statement.

### Order By

```sql
select
	Name
	,MemberID
	,ClaimID
	,Gender
	,case when Age <= 10 then 'Baby'
	  	  when Age <= 20 then 'Young Adult'
	  	  when Age <= 30 then 'Adult'
	  	  else 'Senior' end as AdultClass
from [server_name].[database_name].[table_name]
order by
	Name asc
	,MemberID desc
	,case when Gender = 'Male' then 2
		  when Gender = 'Female' then 1
		  else 0 end desc
	,AdultClass
```

Order by statements are placed at the end of the query and the statement reorders the table based on the given columns. Keywords **asc** and **desc** can be specified after the end of the each column name to specify whether an ascending (or descending) ordering will be applied to that column to the table.

> A case when statement can also be used under the order by statement **provided that** all the columns under the case when statement are under the select statement (In the example above: the column Gender under the case when statement are also selected under the select statement).

> Order by statements can also use aliases unlike the group by statements (in some SQL versions as discussed above).

### Joins

There are 4 types of joins: left joins, right joins, inner joins and full outer joins.

Example:

```sql
select
	a.*
	,b.MemberID as Tbl2MemberID
	,c.Name as Tbl3Name
	,d.Name as Tbl4Name
from [server_name].[database_name].[table_name] a
left join [server_name].[database_name].[table_name2] b
	on a.MemberID = b.MemberID
	   and a.Age <= 10
	   or b.IncurredDate between '2019-01-01' and '2019-01-31'
	   and case when a.Age between 10 and 15 then 1 else 0 end 
	   	   = case when b.Age between 15 and 20 then 1 else 0 end
inner join [server_name].[database_name].[table_name3] c
	on  a.Name = c.Name
		b.MemberID = c.MemberID
		and a.Gender <> 'Male'
full outer join [server_name].[database_name].[table_name4] d
	on c.Name = d.Name
where
	(c.Name is not null or d.Name is not null)
```

Table aliases are commonly used when joining multiple tables. Columns that have the same name across multiple tables have to be referred with their table aliases when selecting multiple columns (i.e. c.Name and d.Name when trying to select both columns from the different tables and the different columns have to be aliased as well to avoid duplicate names when selecting).

The example above shows how the different joins can be used and how the different logical operators (AND or OR) can be used when joining on the keys.

To simply explain the different joins, consider the following 2 tables:

Table A

|  ID  | Num  |
| :--: | :--: |
|  1   | 100  |
|  2   | 200  |
|  3   | 300  |

Table B

|  ID  | Num  |
| :--: | :--: |
|  2   | 200  |
|  4   | 400  |
|  6   | 600  |

Table A **left join** Table B based on ID will result in:

| ID A | Num A | ID B | Num B |
| :--: | :---: | :--: | :---: |
|  1   |  100  | NULL | NULL  |
|  2   |  200  |  2   |  200  |
|  3   |  300  | NULL | NULL  |

A **left join** will return **all** the results on the left table and will only return the corresponding rows from the right table that matches with the join keys. In the above example, only the ID: 2 has a match in both tables, but the left table (Table A) returns all the values where as the right table (Table B) only return the corresponding with the matching value (in this case ID: 2). A **right join** behaves similarly, only in the opposite way.

Table A **inner join** Table B based on ID will result in:

| ID A | Num A | ID B | Num B |
| :--: | :---: | :--: | :---: |
|  2   |  200  |  2   |  200  |

An inner join will **only return where both tables have the matching join keys**. In the above example, since only ID: 2 has value in both the left and right tables only that corresponding ID is returned, all the other records are discarded. Note that writing just **join** instead of **inner join** will result in the same operation (i.e. join defaults to inner join).

Full Outer Join (Full Join)

Table A **full outer join** Table B based on ID will result in:

| ID A | Num A | ID B | Num B |
| :--: | :---: | :--: | :---: |
|  1   |  100  | NULL | NULL  |
|  2   |  200  |  2   |  200  |
|  3   |  300  | NULL | NULL  |
| NULL | NULL  |  4   |  400  |
| NULL | NULL  |  6   |  600  |

A full join will return **all records** from both tables and will match certain records (if applicable) based on the join keys. Mismatched records will have values of **NULL**. Full outer join can be used to 

Re-iterating the example shown above:

```sql
select
	a.*
	,b.MemberID as Tbl2MemberID
	,c.Name as Tbl3Name
	,d.Name as Tbl4Name
from [server_name].[database_name].[table_name] a
left join [server_name].[database_name].[table_name2] b
	on a.MemberID = b.MemberID
	   and a.Age <= 10
	   or b.IncurredDate between '2019-01-01' and '2019-01-31'
	   and case when a.Age between 10 and 15 then 1 else 0 end 
	   	   = case when b.Age between 15 and 20 then 1 else 2 end
inner join [server_name].[database_name].[table_name3] c
	on  a.Name = c.Name
		and b.MemberID = c.MemberID
		and a.Gender <> 'Male'
full outer join [server_name].[database_name].[table_name4] d
	on c.Name = d.Name
where
	(c.Name is not null or d.Name is not null)
```

The example may not have any inherent value, it merely showcases the different possibilities when joining multiple tables together.

> *Case when* statements can also be used under the **on** statement

> **Inner join** can also be used as a *pseudo where statement*. In the example above, the statement:
>
> ```sql
> inner join
> 	and a.Gender <> 'Male'
> ```
>
> guarantees that the resulting final table will exclude the *Male* gender even though only the Gender column from table A is specified. This is due to the fact that inner join only return results where both tables have matching keys. Hence, if the Gender column from table A is excluding the *Male* gender, table B will follow suit.

> In contrast, when doing **left/right joins** a single evaluation on the column of a specific table will only affect that table. For example, using the above example, the statement:
>
> ```sql
>  left join  
>  	and a.Age <= 10
> ```
>
> will only include rows from the left table that has the Age of less than 10. The Age column on the right table will not be evaluated. The join will filter the left table to all records with the Age of less than 10 and will try to join that table with the right table which includes all Ages (which is different than the inner join). **Full outer joins** behave similarly with left/right joins, in that the columns from each tables are evaluated separately.

> Due to how Full outer joins works, the different columns from both tables usually need to be evaluated separately (and with care). For example, the where statement in the example above (table C full outer join table D):
>
> ```sql
> where
> 	(c.Name is not null or d.Name is not null)
> ```
>
> Notice that both c.Name and d.Name are evaluated since full outer joins return **all records** from both tables regardless if there are any matches on the join keys or not.

One to Many Joins, Many to One Joins, Many to Many Joins

In certain cases, a supposedly unique column that will be used for the join keys might have duplicates. 

In the case of a many to one joins, the duplicated values on the secondary table will result in a duplicated records in an otherwise clean primary table.

Consider the following 2 tables:

Table A:

|  ID  | Num  |
| :--: | :--: |
|  1   | 100  |
|  2   | 200  |
|  3   | 300  |

 Table B:

|  ID  | Num  |
| :--: | :--: |
|  1   | 100  |
|  1   | 100  |
|  2   | 200  |

Notice the duplicated record on table B (second row).

Running the following SQL statement will result in duplicated records in table A (in which it should have been clean before joining with table B).

```sql
select
	a.ID
	,a.Num
	,b.ID as b_id
	,b.Num as b_num
from table_a a
left join table_b b
on
	a.ID = b.ID
```

| ID A | Num A | ID B | Num B |
| :--: | :---: | :--: | :---: |
|  1   |  100  | 100  |  100  |
|  1   |  100  | 100  |  100  |
|  2   |  200  |  2   |  200  |
|  3   |  300  | NULL | NULL  |

The second row is now duplicated due to the duplicate values in table B! A similar problem will occur when doing a right join and the secondary table (left table) has several duplicated records and the primary table (right table)does not.

The same problem will also occur when doing **inner joins**, the resulting table if the SQL statement above were to be repeated with an inner join would be:

| ID A | Num A | ID B | Num B |
| :--: | :---: | :--: | :---: |
|  1   |  100  | 100  |  100  |
|  1   |  100  | 100  |  100  |
|  2   |  200  |  2   |  200  |

Again, the second row is now duplicated due to the duplicate values in table B. This is called a **many to one join** and is often not desirable.

On the opposite end, a **one to many join** can also occur, however, in most cases, these are intended:

Take an example:

Table A:

|  ID  | Product ID |
| :--: | :--------: |
|  1   |     1      |
|  1   |     2      |
|  2   |     2      |

Table B:

|  ID  |  Name  |
| :--: | :----: |
|  1   | Andrew |
|  2   |  John  |

```sql
select
	a.ID
	,a.ProductId
	,b.Name
from table_a a
left join table_b b
on
	a.ID = b.ID
```

The join example above is appropriate and very common, after all, this is the main advantage of a relational database. The resulting table will show (same result with an **inner join**):

|  ID  | Product ID |  Name  |
| :--: | :--------: | :----: |
|  1   |     1      | Andrew |
|  1   |     2      | Andrew |
|  2   |     2      |  John  |

Finally, a **many to many join** is usually almost never intentional and will require appropriate data deduplication before proceeding.

Deduplication with Window Functions

Often times, records contain duplicate values that are not intended. Window functions (specifically row_number()) can be used to quickly get rid of the duplicated value.

```sql
select
	*
from (
select
	MemberID
	,OrderID
	,timestamp
	,row_number() OVER (PARTITION BY MemberID ORDER BY timestamp desc) as rn
from [server_name].[database_name].[table_name]
    )
where 
	rn = 1
```

The above example shows an extremely common way to deduplicate a column (multiple combinations of columns can also be used by specifying the combination under the PARTITION BY statement in the window function).  The above example will attempt to give a number to each of the row ordered by, in this case, timestamp in a descending order. If multiple rows have the same OrderIDs (which should have been unique) the row_number() will assign the values 1, 2, ... and so on. Finally, the 

```sql
where 
	rn = 1
```

filter at the end only include the first records in which it has the latest timestamp (since it will only include records where the row_number() returns the value of 1)

For further example:

| MemberID | OrderID | timestamp | row_number() |
| :------: | :-----: | :-------: | :----------: |
|    1     |    1    |     1     |      1       |
|    1     |    2    |     3     |      1       |
|    1     |    2    |     2     |      2       |
|    1     |    2    |     1     |      3       |
|    1     |    3    |     2     |      1       |
|    2     |    1    |     3     |      1       |
|    2     |    2    |     1     |      1       |
|    3     |    1    |     5     |      1       |

Note that the row_numbers() reset to 1 for each MemberID (even though they have the same OrderID) due to the data

```sql
OVER (PARTITION BY MemberID ...
```

being partitioned by the MemberIDs. Regardless of that, notice that the value of the column row_number() on the duplicated OrderIDs 1 of MemberID 1 changes from 1 to 3 depending on the chosen ORDER BY statement (in this case it was ordered by the descending order of the timestamp column (latest data)). By labeling it this way, and finally only including where the row_number() = 1, it is a very efficient and simple way to deduplicate a table.

The same deduplication method is commonly used when joining multiple tables which have duplicates themselves (the method may be done before (resulting in different subqueries for cleaning up the individual tables prior to joining) or after the join).

Window functions will be discussed in further detail in a different section below.

Self-Joins

Self-Joins are rarely used with the addition of window functions. Self joins are much slower than window functions, however they still do have some merits. The most classic example of a self-join will be to calculate rolling sums, take a look at the following example:

Table A:

| Day  | Visitors |
| :--: | :------: |
|  1   |    10    |
|  2   |    20    |
|  3   |    30    |

We can do the following to calculate the rolling sum of visitors across the different days (left join instead of an inner join will achieve the same result as well):

```sql
select
	a.Day
	,a.Visitors
	,b.Day as b_day
	,b.Visitors as b_visitors
from [server_name].[database_name].[table_A] a
inner join [server_name].[database_name].[table_A] b
on
	a.Day >= b.Day
```

Notice the same table that we are joining together (with different aliases) and the above query will return the following table:

| Day A | Visitors A | Day B | Visitors B |
| :---: | :--------: | :---: | :--------: |
|   1   |     10     |   1   |     10     |
|   2   |     20     |   1   |     10     |
|   2   |     20     |   2   |     20     |
|   3   |     30     |   1   |     10     |
|   3   |     30     |   2   |     20     |
|   3   |     30     |   3   |     30     |

Notice that the column **Visitors B** now contains all the visitors on a given Day A and prior. For an example, take a look at the records where Day A has a value of 3: there are 3 different records on the Day B column: 1, 2 and 3 corresponding to that given Day A and prior. Finally, we can now wrap the query above in another query and group by the column Day A to calculate the rolling sum:

```sql
select
	Day
	,sum(b_visitors) as rolling_sum
from (
select
	a.Day
	,a.Visitors
	,b.Day as b_day
	,b.Visitors as b_visitors
from [server_name].[database_name].[table_A] a
inner join [server_name].[database_name].[table_A] b
on
	a.Day >= b.Day
      )
group by
	Day
```

And the resulting table (grouped by Day A):

| Day A | sum(Visitors B) |
| :---: | :-------------: |
|   1   |       10        |
|   2   |       30        |
|   3   |       60        |

The same result above can be achieved using Window functions which are more efficient and simpler to use than using self joins:

```sql
select
	Day
	,sum(Visitors) OVER (ORDER BY Day) as rolling_sum
from [server_name].[database_name].[table_A]
```

Again, with the addition of window functions, self-joins are now rarely used as window functions can achieve the same result with much less effort and with more efficiency.

### SQL Statement Orders

Before moving on to the next section, the keen reader might have already noticed that the different SQL statements need to be ordered in a certain order for the script to work. In particular:

```sql
--[ <INSERT INTO clause> ] can be used in lieu of the <into clause>
select
	...
into [ <into clause> ]
from [server_name].[database_name].[table_name]
join
	[ <join clause> 
      on ...		]
where
	[ <where clause> ]
group by
	[ <group by clause> ]
having 
	[ <having clause> ]
order by
	[ <order by clause> ]
```



### Temporary Tables (Local & Global)

Temporary tables are indicated by **#**. For example:

```sql
select *
from #local_temp_table
```

The **#** indicates the table is temporary and will be automatically deleted once the session/connection to the database turns off.

A global temp table are indicated by **##**. For example:

```sql
select *
from ##global_temp_table
```

Global temporary tables are more permanent than temporary tables and will remain until the database is turned off (unlike local temporary tables where it will be deleted when a single computer connection to the database that created the temporary table is turned off). Global temporary tables can also be accessed by multiple different connections to the database, unlike local temporary tables where only the computer that made the local temporary table can access the table.

```sql
DROP TABLE ##global_temp_table
DROP TABLE #local_temptable

IF OBJECT_ID('tempdb..#temp') IS NOT NULL
    DROP TABLE #local_temp_table
```

The DROP TABLE functions can be called to forcefully delete the temporary tables. Temporary tables reside under **'tempdb..'**

Temporary table creation can be done using the **into** statement or using a **CREATE TABLE** statement which will be discussed in a different section below.

### Subqueries

Subqueries are queries wrapped in another query. For example:

```sql
select
	a.MemberID
	,a.Name
	,b.MedicaidClass
from [server_name].[database_name].[table_A] a
left join
	--this is a subquery
	(select distinct
    		MemberID
    		,MedicaidClass
   	 from [server_name].[database_name].[table_b]) b --notice the table alias placed OUTSIDE of the parantheses
   	 --end subquery
   	 
on
	a.MemberID = b.MemberID
```

In the example above, the subquery is used to deduplicate the MemberID and MedicaidClass values in table B before joining to table A to avoid a many to one join. Take a look at another example:

```sql
select top 100
	State
	,TotalVisitors
from (
    --this is a subquery
	select
    	 Day
    	 ,State
    	 ,sum(Visitors) as TotalVisitors
    from [server_name].[database_name].[table_A]
    group by
    	Day
    	,State
    -- end subquery
	) --table alias is not needed since joins are not being done
where
	Day = 'Thursday'
order by
	TotalVisitors desc
```

The subquery above return the total number of visitors on a given Day and State. The subquery is wrapped in another query that returns the top 100 state that has the most visitors on a Thursday.

Subqueries are often used to simplify a complicated query and can break it down into several smaller steps. Subqueries can often be nested, however, deep nested levels of subqueries will affect readability and is ill-advised (for production codes). Instead, CTEs (which will be discussed below) can be used as a solution that will improve code readability with similar functionalities. The advantages that subqueries have over CTEs are that they are shorter to write (great for scripting) and are sometimes reported to be a tiny bit faster.

Subqueries can also be used as a filter in the where statement (accompanied with *in*) as illustrated by one of the examples above, but to re-iterate:

```sql
select
	Name
	,MemberID
from [server_name].[database_name].[table_a]
where
	Class in (select
              		MedicaidClass
               from [server_name].[database_name].[table_b]) --table alias not needed
```

Notice the two different tables the query references to. A select distinct can be used in the second table (table B) when selecting the MedicaidClass (in the example above), although, not necessary, hence it is left out in this example (since data duplications are not an issue due to the goal of the where filter is just to filter the records based on a list available in table B). However, keep in mind that when doing filters with subquery the select statement from the second table (table B) can only return 1 column (users may choose to concatenate different columns to make a key and filter on that instead if users need to filter on multiple columns at one time).

### Variables

Variables have to be declared in MSSQL with the **DECLARE** statement. Variables are indicated by the symbol **@** in front of the variable name. The statement **set** is used to set the value of the variable.

```SQL
DECLARE @MinIncurredDate datetime
DECLARE @Name varchar(max)
Set @MinIncurredDate = '2019-01-01'
Set @Name = 'Johnny'
```

 A datatype must also be specified when declaring a variable.

### Data Types

These are the several most commonly used datatypes in SQL:

|         Data Type         |                         Description                          |
| :-----------------------: | :----------------------------------------------------------: |
|          bigint           |                 integer from -2^63 to 2^63-1                 |
|            int            |                 integer from -2^31 to 2^31-1                 |
| decimal(precision, scale) |       numeric data type with fixed precision and scale       |
|           money           | financial data type from -2^63 to 2^63-1 with the precision of one ten-thousandth unit. |
|         float(n)          |            numeric data type with float precision            |
|           real            | numeric data type with float precision that is defined as a float(24) |
|         datetime          |             data type representing date and time             |
|          char(n)          |  text string of fixed length (maximum length of 8000 chars)  |
|        varchar(n)         | text string of variable length (maximum length of 2^31-1 chars) |
|        nvarchar(n)        | unicode text string of variable length (maximum length of 4000 chars) |
|           text            | text string of variable length (maximum length of 2^31-1 chars) |
|           ntext           | unicode text string of variable length (maximum length of 2^30-1 chars) |

> Note: varchar() and nvarchar() can be declared as varchar(max) or nvarchar(max) to allow the maximum length

**cast()** functions can be used to convert different data types to another. For example:

```sql
select cast(10 as varchar(10))
```

Will convert the integer data **10** to a varchar data type of length 10.

### Control Flows (Creating Re-executable SQL Scripts)

IF EXISTS

Often times, users create production SQL scripts that are run everyday. If the script involves writing the result into a table, rerunning the script again will result in an error if the table already exists. The IF EXISTS statement can check if an object already exists in the database and can alter/drop the table if it already exists. For example:

```SQL
IF EXISTS (SELECT * FROM DW_Staging..sysobjects 							
		   WHERE id = OBJECT_ID(N'DW_Staging.[XXXXXXX\niko.hiananto].RunningClaims') )
BEGIN
	Drop table DW_Staging.[XXXXXXX\niko.hiananto].RunningClaims
END

--Your Query Here:
select
	*
into DW_Staging.[XXXXXXX\niko.hiananto].RunningClaims
from DW_Staging.[XXXXXXX\niko.hiananto].RawClaimsTable
GO
```

In the Example above, the IF EXISTS statement first check whether or not the table already exists. If it did, the table will be dropped and the query below can be executed without worrying that the query will error out due to the existence of the same table. **BEGIN** and **END** statements can be used to make sure that everything inside that code block will be executed after the IF statement.

GO

The GO statement will force the script to run the block of code above the GO statement before proceeding to a different part of the script. The GO statement is useful when working with long SQL production scripts that rely on a lot of control flows and table alterations (IF EXIST statements, table drop statements), that way the flow of the script is evaluated properly and the script will run as intended.

WHILE Loops

While loops behave just exactly as you would expect. It is often used to append multiple tables, into a single table or re-execute a query multiple times. As the current of the writing of this reference text, FOR loops are not available in MSSQL Server, instead while loops are used to replace the functionality of a for loop. For example:

```sql
DECLARE @i int
Set @i = 1
WHILE @i <= 20
BEGIN
--Execute query
INSERT INTO DW_Staging.[XXXXXXX\niko.hiananto].RawClaimsTable
select
	Name
	,MemberID
	,ClaimsID
from DW_Staging.[XXXXXXX\niko.hiananto].RawClaimsTableDaily

Set @i = @1 + 1
END
```

Again, the BEGIN and END block can be used to run multiple queries before proceeding with the loop.

Control flows differ largely depending on the SQL platform and as such will only be covered briefly.

### Common Table Expressions (CTE)

Common Table Expressions are commonly used to replace subquery to improve code readability and to break down complicated queries into several smaller steps. They function exactly the same as a subquery (or a temporary table for that matter) and are used to help break down large queries into several, more manageable, queries:

```SQL
WITH CTE as (
select
	Name
	,MemberID
from [server_name].[database_name].[table_a]
where
	Gender = 'Male'
), your_cte_name_here as ( --CTEs can be nested
select
    MemberID
    ,MedicaidClass
from [server_name].[database_name].[table_b]
where
    MemberID in (select 
                 	MemberID 
                 from CTE)
)
select
	a.MemberID
	,a.ClaimID
	,b.MedicaidClass
from [server_name].[database_name].[table_c] a
left join your_cte_name_here b
on
	a.MemberID = b.MemberID


--Illegal:
select
	*
from CTE
```

As the example above shows, CTE works similarly to temporary tables and starts with the **WITH ... as (** statement, where the ... represents the CTE name that user can freely choose. CTEs can be nested as shown above and can also be used within a subquery. CTEs are faster than temporary tables (and preferable if the query performing on the resulting table will only be evaluated once) since CTEs do not need to write the resulting query on a table. Temporary tables are still preferable if the resulting query will be used multiple times in different queries later on, since CTE results can only be used on that given WITH statement only. The next set of queries referring to that CTE name (outside the WITH statement) will return an error, as shown by the example above:

```sql
--Illegal:
select
	*
from CTE
```

Since it is done after the call of the with statement is finished, the table has now been erased and the result can no longer be retrieved. Any select statements referring to the table created after that WITH statement is no longer appropriate.

CTEs have another use and that is to return **hierarchical queries using recursive CTEs**. Recursive CTEs involve the use of an anchor query statement and a caller. Further discussion of the recursive CTE is outside the scope of this text.

### Table Unions

Multiple tables can be appended to each other by the use of the union function. Two types of union functions are available: **union and union all**. Union deletes duplicated records (if there are any across the 2 tables) whereas union all stores all the records from both tables. Union all is more commonly used and a data clean up process can be done afterwards, that way users have more control in which records are kept and which are deleted.

```SQL
select
	Name
	,MemberID
from [server_name].[database_name].[table_a]

union all

select
	Name
	,MemberID_B
from [server_name].[database_name].[table_b]

union all

select
	Name
	,MemberID_C
from [server_name].[database_name].[table_c]
...
```

The union operator appends the data records by column, hence, it is the to the **utmost importance** to make sure that the **column order** from each of the select statements **match**. However, the column names do not need to match (only the ordering matters) as illustrated in the example above where the MemberID from table A is appended with MemberID_B column from table B and again with the column MemberID_C from table C. Union all operators can be stacked as many times as desired.

> a Union By Name statement sometimes are available in certain SQL languages. In this case the column names matter and the column orderings do not.

### Table Alterations and Creation

##### CREATE TABLE

The CREATE TABLE statement is used to create a new table in a database. Tables can have certain constraints when creating them: NULL constraints, uniqueness constrains on columns, default values, keys, etc. The discussion of the constraints is outside the scope of this text.

```sql
CREATE TABLE [server_name].[database_name].[table_name] (
    MemberID int,
    LastName varchar(MAX),
    FirstName varchar(255),
    Address varchar(255),
);
```

The create table function can also be used to create temporary tables.

##### DROP TABLE

The DROP TABLE statement is used to drop a table in a database.

```SQL
DROP TABLE [server_name].[database_name].[table_name];
```

##### ALTER TABLE

The ALTER TABLE statement is used to add, delete, or modify columns in an existing table. The ALTER TABLE statement can also be used to add and drop various constraints on an existing table (NULL constraints, uniqueness constrains on columns, default values etc).

```sql
--Add Column
ALTER TABLE [server_name].[database_name].[table_name]
ADD MemberClass varchar(10);

--Drop Column
ALTER TABLE [server_name].[database_name].[table_name]
DROP COLUMN MemberID;

--Change column datatype
ALTER TABLE [server_name].[database_name].[table_name]
ALTER COLUMN BirthDate datetime;

--Rename Column (in MySQL only)
ALTER TABLE [server_name].[database_name].[table_name]
CHANGE MemberID MemberIDNumber int;

--Rename Column (in MSSQL Server)
--Changes the name of a user-created object in the current database.
EXEC sp_rename '[server_name].[database_name].[table_name].MemberID', 'MemberIDNumber', 'COLUMN';

--Rename Table (in MSSQL Server)
EXEC sp_rename '[server_name].[database_name].[table_name]', '[server_name].[database_name].[table_name2]';  
```

##### INSERT INTO

The INSERT INTO statement is used to insert new records in an existing table

```sql
INSERT INTO [server_name].[database_name].[table_name] (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);

--If you are adding values for all the columns of the table, you do not need to specify the columns
INSERT INTO [server_name].[database_name].[table_name]
VALUES (value1, value2, value3, ...);
```

### Indexes

Indexes sorts the table based on a key that makes it faster to run common queries on the indexed key. Indexes do take up storage space and are usually dropped first when appending a lot of data to that table before being added back in after the appending process is finished. The reason is due to that indexed tables take a longer time to write into than non-indexed table. Usually the primary key is used as an index as it is the most common column to be joined across and be filtered on. Filtering on an indexed column is much faster than otherwise, likewise joining on two indexed columns from two different tables are much much faster than otherwise. To create an index a column (or multiple columns) has to be specified for the table to be indexed on. Indexes can be created as clustered indexes or non-clustered indexes.

```SQL
-- Create a nonclustered index on a table or view
CREATE INDEX index_name ON [server_name].[database_name].[table_name] (MemberID);

--Create a clustered index on a table or view
CREATE CLUSTERED INDEX index_name ON [server_name].[database_name].[table_name] (col1);

--Create a unique index on a table or view
CREATE UNIQUE INDEX index_name ON [server_name].[database_name].[table_name] (MemberID DESC, Name ASC, MemberClass ASC);

--Dropping a created index
DROP INDEX [server_name].[database_name].[table_name].index_name;
```

A unique index can be created as clustered or non-clustered and is only allowed in which no two rows can have the same index value combination.

> A clustered index creates an index in which the logical order of the key values determines the physical order of the corresponding rows in a table. The bottom, or leaf, level of the clustered index contains the actual data rows of the table. A table or view is allowed one clustered index at a time.

> A non-clustered index creates an index that specifies the logical ordering of a table. With a non-clustered index, the physical order of the data rows is independent of their indexed order.

### Window Functions

Window functions have now become an integral part of SQL. They are very simple to use yet very efficient and very powerful making them one of, if not. the most useful functions available in SQL. A window function has the following syntax:

```SQL
<FUNCTIONS>
OVER (   
       [ <PARTITION BY clause> ]  
       [ <ORDER BY clause> ]   
       [ <ROW or RANGE clause> ]  
      )  
```

A list of the most commonly used functions are below:

| Function          | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| SUM(expr)         | return the sum of the expression over a window clause        |
| AVG(expr)         | return the average of the expression over a window clause    |
| MIN(expr)         | return the minimum of the expression over a window clause    |
| MAX(expr)         | return the maximum of the expression over a window clause    |
| COUNT(expr)       | return the count of the expression over a window clause      |
| ROW_NUMBER()      | return the numbering of the expression over a window clause  |
| RANK()            | return the rank of the expression over a window clause       |
| DENSE_RANK()      | return the dense rank of the expression over a window clause |
| FIRST_VALUE(expr) | return the first value of the expression over a window clause |
| LAST_VALUE(expr)  | return the last value of the expression over a window clause |
| LAG(expr, n)      | return the previous (n) offset value of the expression over a window clause |
| LEAD(expr, n)     | return the next (n) offset value of the expression over a window clause |

The list is not exhaustive, but contains almost all the most commonly used functions. All of the functions above can be used over a window clause. Unfortunately, at the time of the writing of this document there is no COUNT(DISTINCT ) capability over a window clause, however, there are some workarounds that will be discussed in a section below. The next section will now give an overview of the [PARTITION BY clause]

The PARTITION BY clause divides the query result set into partitions. The window function is applied to each partition separately and **computation restarts** for each partition. It is analogous to the group by statement. Several columns can be included under the PARTITION BY clause.

The ORDER BY clause defines the logical order of the rows **within each partition** of the result set. It specifies the logical order in which the window function calculation is performed. An Order By statement goes in this clause, and as such each column can be specified to be ordered by either in ascending (ASC) or descending (DESC) order.

The ROW or RANGE clause further limits the rows within the partition by specifying start and end points within the window partition. This is done by specifying a range of rows with respect to the current row either by logical association or physical association. 

With the **ROWS** option you define on a ***physical level*** how many rows are included in your window frame. With the **ROWS** option you define a fixed number of rows preceding and following the current row. Which rows you see here through your window frame depends on the **ORDER BY** clause of the window frame. 

With the **RANGE** option how many rows are included in the window frame depends on the **ORDER BY** values. The **RANGE** option includes all the rows within the window frame that have the same **ORDER BY** values as the current row. With the **RANGE** option you define your window frame on a ***logical level***. If more rows have the same **ORDER BY** value, your window frame consists of more rows than when you use the **ROWS** option.

Below shows an example of the full syntax of the window function (and the possibilities):

```SQL
FUNCTION(expr)
OVER (
	 PARTITION BY Column_1, Column_2, Column_3, ...
     ORDER BY Column_1 ASC, Column_2 DESC, Column_3 ASC, ...
     ROWS|RANGE BETWEEN UNBOUNDED PRECEDING|<unsigned integer n> PRECEDING|CURRENT ROW AND UNBOUNDED FOLLOWING|<unsigned integer n> FOLLOWING|CURRENT ROW
	 )
```

For example: the clause ROWS BETWEEN 5 PRECEDING AND 5 FOLLOWING will evaluate all records between 5 records before the current record and 5 records following the current record. ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING will evaluate all records in the dataset at once (given a partition). ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW will evaluate all records from the first row up to the current row (given a partition).

Take an example of a running total. Let's say:

Table A

| Year | Month | Visitors |
| :--: | :---: | :------: |
| 2019 |   1   |   100    |
| 2019 |   2   |   200    |
| 2019 |   3   |   150    |
| 2019 |   3   |   150    |

And the following query is run on the table:

```sql
select
	Year
	,Month
	,sum(Visitors) OVER (ORDER BY Year, Month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) RowsRunningTotal
	,sum(Visitors) OVER (ORDER BY Year, Month RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
RangeRunningTotal
from [server_name].[database_name].[table_A]
```

Notice that records 3 and 4 on table A has the same Year and Month combination and therefore the same ORDER BY value combination. The above query will return the following result:

| Year | Month | Visitors | RowsRunningTotal | RangeRunningTotal |
| :--: | :---: | :------: | :--------------: | :---------------: |
| 2019 |   1   |   100    |       100        |        100        |
| 2019 |   2   |   200    |       300        |        300        |
| 2019 |   3   |   150    |       450        |        600        |
| 2019 |   3   |   150    |       600        |        600        |

Since the window is defined to be between UNBOUNDED PRECEDING and the CURRENT ROW, the OVER clause returns a running total sum that adds up all the rows from the first row up to the current row.

Notice that records 3 and 4 on the resulting query have differing running total. This is due to the RANGE option defining the window frame on a logical level. Since the Year and Month combination on records 3 and 4 have the same ORDER BY values, they are counted as a single row (even though physically they are 2 rows). Hence, the window clause BETWEEN UNBOUNDED PRECEDING and the CURRENT ROW defines a window from the first row up to the current row (which the current row on row 3 includes row 4 and, similarly, the current row on row 4 includes row 3). 

The ROW option on the other hand defines the window frame on a physical level. Even if the Year and Month combination on records 3 and 4 have the same ORDER BY values, they are still treated as separate different rows (since they are physically different rows). Hence, the window clause BETWEEN UNBOUNDED PRECEDING and the CURRENT ROW defines a window from the first row up to the current *physical* row. This might also pose a different problem, as the ORDER BY values are the same, every time the query is run, the ordering between records 3 and 4 **may swap**! This may cause *different query results every time the query is run*. In the example above, it does not matter since there are no other columns, but keep this problem in mind when defining the ORDER BY clause and when using the ROW option.

The image below also illustrates the window frames being evaluated across the ID and value column (PARTITION BY ID). 

![Picture1_WindowingFunctions](C:\Users\Niko Hiananto\Desktop\Picture1_WindowingFunctions.png)

As discussed previously, a distinct count of a column over a particular window can be done with the use of sum(expr) and the row_number(expr). Suppose the table in the following example contains duplicates under the MemberID column and ClassID column and suppose that we need to count the number of distinct MemberID by State as well as the number of distinct combination of MemberID and ClassID. Then a query below will suffice:

```sql
select
	State
	,sum(UniqueRnMemberID) as DistinctMemberIDCount
	,sum(UniqueRnMemberIDClassID) as DistinctMemberIDClassIDCount
from
	(
    select
        State
        -- Unique Across MemberID
        ,case when 1 = row_number() OVER (PARTITION BY MemberID ORDER BY timestamp desc) 
        	  else 0 end as UniqueRnMemberID
        --Unique Across MemberID and ClassID
        ,case when 1 = row_number() OVER (PARTITION BY MemberID, ClassID ORDER BY timestamp desc) 
        	  else 0 end as UniqueRnMemberIDClassID
    from [server_name].[database_name].[table_name]
     )
group by
	State
```

> the case when statement will assign a value of 1 if the row_number() function returns 1 and will return 0 otherwise, allowing the user to count the number of distinct values for a given PARTITION by summing up the resulting column.

> the OVER clause ORDERS the window frame to defined by the descending order of the column timestamp (latest data), meaning that the value of 1 will be assigned to the latest data that came in for a given PARTITION and a value of 0 otherwise.

The downside of using this method is that the ordering does matter, in that, only one record will have a value of 1 (all other duplicates will have a value of 0 based on the order by clause) and the resulting column will be summed across. This is usually not a problem, but keep in mind that sometimes in real-world applications duplicated records might have one true real value while the other records have wrong values in the other columns except the ID columns (only duplicated in the ID column).

### Dynamic SQL

Dynamic SQL can uses varies greatly. A more commonly used aspect of Dynamic SQL is to pass on a variable where filter where the criteria changes dynamically depending on the user. A detailed explanation on how to use Dynamic SQL will be omitted, instead an example of a use of Dynamic SQL will be shown below:

```SQL
/*********************************************													
Combine Overlapping Date Ranges															
**********************************************/													
Declare @sql Varchar(max)															
Declare @TempTable varchar(max)															
Declare @FinalTable varchar(max)															
Declare @i int															
Declare @outsql nvarchar(max)															
Declare @output2 int															
															
Set NOCOUNT ON															
															
Set @i = 1															
Set @output2 = 0															
/*Check if there are any overlaps*/															
			Set @TempTable = concat('##LTC_', @i)												
															
			Set @outsql = 												
					'										
					(select @output = count(*)										
					from ' + @TempTable + ' A										
					inner join ' + @TempTable + ' B										
					on A.MemCIN = B.MemCIN and										
						((B.MemBeginDate between A.MemBeginDate and A.MemEndDate)									
					or (B.MemEndDate between A.MemBeginDate and A.MemEndDate))										
					where (A.MemBeginDate <> B.MemBeginDate and A.MemEndDate <> B.MemEndDate))										
				 '											
			EXEC sp_executesql @outsql, N' @output int out', @output2 OUT												
															
while @output2 <> 0															
BEGIN															
Set @TempTable = concat('##LTC_', @i)															
Set @FinalTable = concat('##LTC_',@i + 1)																										
/*Combine Overlapping date ranges recursively*/														
	Set @sql = '														
	drop table ' + @FinalTable + '														
	;WITH CTE AS(														
		select													
			MemCIN												
			,MemBeginDate												
			,MemEndDate												
			,Grp1 = case	when (MemBeginDate not between 											
			LAG(MemBeginDate) over (partition by MemCIN order by MemBeginDate ASC) and LAG(MemEndDate) over (partition by MemCIN order by MemBeginDate ASC))												
					or LAG(MemBeginDate) over (partition by MemCIN order by MemBeginDate asc) is null										
					then row_number() over (partition by MemCIN order by MemBeginDate asc)										
							else NULL --If date ranges overlaps assign NULL VALUE								
							end								
		from ' + @TempTable + '													
	), CTE2 as(														
	select														
		*													
		,Grp2 = max(Grp1) OVER (PARTITION BY MemCIN ORDER BY MemBeginDate asc)													
	from CTE														
	)														
	select														
		MemCIN													
		,MemBeginDate = min(MemBeginDate)													
		,MemEndDate = max(MemEndDate)													
	into ' + @FinalTable +														
	'														
	from CTE2														
	group by														
		MemCIN													
		,Grp2													
			'												
EXEC(@SQL)															
															
/*continue joining until no overlaps*/															
		Set @outsql =													
				'											
				(select @output = count(*)											
				from ' + @FinalTable + ' A											
				inner join ' + @FinalTable + ' B											
				on A.MemCIN = B.MemCIN and											
					((B.MemBeginDate between A.MemBeginDate and A.MemEndDate)										
				or (B.MemEndDate between A.MemBeginDate and A.MemEndDate))											
				where (A.MemBeginDate <> B.MemBeginDate and A.MemEndDate <> B.MemEndDate))											
			 '												
		EXEC sp_executesql @outsql, N' @output int out', @output2 OUT																									
															
Set @i = @i +1															
END															
```

In short, a text variable is used to represent the SQL query and the text will be executed with an **EXEC(@sql)** statement where the @sql is a text variable containing the SQL script. An **EXEC sp_executesql** is sometimes preferable (to avoid SQL injection) and is a must if the user needs to output a variable from the execution of the dynamic SQL. Since, the use of Dynamic SQL is scarce in, nowadays, cloud databases the discussion of this section will be kept brief.

### Common Functions

In this section the most commonly used functions will be listed:

##### Aggregate Functions

Aggregate functions require the use of a group by statement

| Function                   | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| sum(...)                   | takes the sum of a column                                    |
| count(...)                 | counts the number of records in that column                  |
| count(distinct ...)        | count the unique number of records in that column            |
| avg(...)                   | takes the average of a column                                |
| min(...)                   | returns the minimum value in that column                     |
| max(...)                   | returns the maximum value in that column                     |
| string_agg(..., separator) | concatenates values of string expressions and places a separator in between |

##### NULL Functions

These functions involve dealing with NULL values in a dataset

| Function                  | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| NULLIF(... , ...)         | returns a different column if the column in the first argument returns NULL |
| coalesce(...,...,...,...) | ANSI standard (available in most SQL languages), behave the similarly as a NULLIF function, however multiple columns can be passed on to the argument. |

##### Case When Functions

| Function                           | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| case ... when ... then             | evaluates the column multiple times and return different results depending on the criteria |
| IIF(bool, true_value, false_value) | can be used as a short hand for the case when statement when only evaluating one condition. |

##### String Functions

| Function                              | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| LTRIM(expr)                           | deletes white spaces on the left of the expression           |
| RTRIM(expr)                           | deletes white spaces on the right of the expression          |
| TRIM(expr)                            | deletes white spaces on the left and right of the expression |
| SUBSTRING(expr, start, length)        | takes the substring of a string                              |
| RIGHT(expr, integer)                  | takes the first n characters from the right                  |
| LEFT(expr, integer)                   | takes the first n characters from the left                   |
| CONCAT(arg1,arg2,...)                 | concatenate multiple strings together                        |
| CONCAT_WS(separator, arg1, arg2, ...) | concatenate multiple strings together with a separator       |
| REPLACE(expr, pattern, replacement)   | replace a pattern in a given string with a different value   |
| REVERSE(expr)                         | reverses the characters in a string                          |
| LOWER(expr)                           | returns the string with all lowercase                        |
| UPPER(expr)                           | returns the string with all uppercase                        |

> REPLACE function can be used reliably to delete all white spaces by doing REPLACE(string_expression, ' ' ,'').

##### Date Functions

| Function                                   | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| YEAR(date)                                 | returns an integer that represents the year of the specified date |
| MONTH(date)                                | returns an integer that represents the month (month of the year) of the specified date |
| DAY(date)                                  | returns an integer that represents the day (day of the month) of the specified date |
| DATEDIFF (datepart, start_date,  end_date) | returns the count (as a signed integer value) of the specified datepart boundaries crossed between the specified start_date and end_date |
| DATEADD (datepart, number , date)          | adds a specified number value (as a signed integer) to a specified datepart of an input date value and returns that value. |
| DATEFROMPARTS (year, month, day)           | returns a date value that maps to the specified year, month, and day values |
| EOMONTH(start_date, month_to_add)          | returns the last day of the month containing a specified date |

### Further Reading

Several other topics are not covered in this text (or are only covered briefly) and may be useful for some of the readers:

| Topics                               | Brief Description                                            |
| :----------------------------------- | :----------------------------------------------------------- |
| Indexes                              | speeds up querying on that table                             |
| Recursive CTEs                       | used to query hierarchical data                              |
| Database Triggers                    | triggers an action when given a certain action to the database |
| Table Valued Functions               | returns a table given an argument (can return different tables depending on the argument given) |
| Dynamic SQL                          | can be used to design queries that changes on the fly (search filters on search engine queries, etc) |
| String Aggregation with For XML Path | can be used to aggregate multiple rows of string into a single record (if string_agg functionality is not available) |

