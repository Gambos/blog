---
title: "SQL Optimisation Cheetsheet"
date: 2025-11-17
draft: false
tags: ["Data Engineer","SQL"]
description: "How to optimise your SQL "
showTableOfContents: true
---
## 1.	How to optimise SQL -- Use Explain plan
### a)	What is the Explain Plan?

An execution plan describes how the database engine executes a SQL statement — the path it takes through indexes, joins, and tables. It can be overridden by using a hint in query.

### b)	How to read and use the Explain Plan
1)	Cost: An estimated measure of resource consumption (lower is better, but this is not absolute).
2)	Access path: Whether the engine uses a full table scan, index scan, or nested loop.
3)	Join methods: Nested loop, hash join, or merge join — each has different performance implications depending on data size and indexing.
4)	Order of operations: The execution starts from the most indented line upward, this is also reflected in the sequence number.
By comparing the plan to your intended logic, you can identify inefficiencies such as unnecessary full table scans or expensive join orders. This is also an extremely useful view to help you recognise if you are on the right track when you are adjusting and optimising your query.

## 2.	Typical Issues for non-optimised SQL and solutions 

### a) Poorly written SQL Statement 
1)	Using SELECT * from large table
Bad example 🚫 Do you really need them all?
 
Good example ✅Only take the columns needed for transformation/migration 
2)	Use ‘LEFT JOIN + WHERE condition IS NOT NULL’ instead of INNER JOIN
Bad example 🚫 If in table departments there is no null dept_name value, 
Good example ✅ then the query above is equivalent to more efficient INNER JOIN 
 
3)	Join same table multiple times/Create multiple very similar CTEs and join on them
Bad example 🚫 
 
Good example ✅ Use CASE WHEN and GROUP BY instead, try to cut down the level of aggregation:

4)	Inefficient JOIN: not utilize different cardinality of tables
Cardinality refers to the number of distinct values in a column:
- High cardinality: The column has many unique values, and each value appears only a few times. Examples: customer_id, order_id, email;
- Low cardinality: The column has only a few distinct values, and many rows share the same value. Examples: country_id, gender, status.

When a low-cardinality column is used in a WHERE or JOIN condition, it has weak filtering power — even if it has an index, the database might still scan a large portion of the table. In contrast, a high-cardinality column provides strong selectivity and allows the optimizer to quickly narrow down the relevant rows.

When the database decides the join order, it prefers to process the table that can reduce the data volume as early as possible—so tables with high cardinality and high selectivity should be used as the driving tables, and tables with low cardinality (such as country or status) should be joined later, because they tend to produce large intermediate result sets.

That’s why we need to keep in mind that the optimizer might assume that starting the hash join from the small table country is cheaper (because the table itself is small), but it overlooks the fact that its low cardinality causes the query to scan the entire orders table—that’s why we need to check explain plan to see if it chooses full scan in this case.

Bad example 🚫 If table country only has a few distinct values (low cardinality) and without any filtering power, the join may start the join from table country and produces huge intermediate sets and forces full scan of table order.
 
Good example ✅ If possible, apply a filter, and the optimizer can estimate the selectivity more accurately, as it realizes that only part of orders belongs to ‘US’, and it can use the index (if added) on o.country_id to limit the scan of large table order.
 
5)	Overuse DISTINCT
Bad example 🚫 DISTINCT masks duplicate logic errors from bad join, but the engine still processes all rows before deduplication — heavy sort or hash step.
 
Good example ✅ Identify and remove duplication at the source (e.g., incorrect joins or UNIONs) rather than applying DISTINCT as a patch.
 
6)	Complex subquery
Bad example 🚫 Inefficient nested subqueries
 
Good example ✅ Fix by using joins or CTE
 
7)	Using NOT IN instead of NOT EXISTS
Bad example 🚫 If the subquery returns 1 NULL, the entire where condition fails (if one value ever equals to NULL? SQL considers it as unknown), so no row returned for the whole query.
 
Good example ✅ In EXISTS, each comparison is evaluated row by row using the = operator. Since NULL is not equal to any value (including another NULL), such rows never satisfy the match condition, so the result will not be disrupted by NULL value.
 
Example matrix for understanding (if employees.dept_id = 30):
Subquery dept_id values	NOT IN result	NOT EXISTS result
{10, NULL}	filtered out	returned
{10, 20}	returned	returned

### b)	Index Missing or Inefficient Usage
Indexes are one of the most important tools to improve SQL performance. They allow the database to locate rows quickly without scanning the entire table.
Issue	Explain plan Key Phrase	Why it’s bad	How to Fix it	Before Example	After Example
Completely Missing Index	TABLE ACCESS FULL	Full table scan instead of index scan will slow down the performance.	Create index on columns which used in:
-  Where
- Join 	SELECT * FROM employees WHERE emp_id = 123;	CREATE INDEX emp_id_idx ON employees(emp_id);

SELECT * FROM employees WHERE emp_id = 123;
Composite Index Missing	TABLE ACCESS FULL;
INDEX RANGE SCAN on partial column	Result in suboptimal query performance as it does not leverage multiple single-column indexes effectively.	Create a composite (multi-column) index matching the filter or join condition order.	CREATE INDEX orders_cust_date_idx ON orders(customer_id)

SELECT * FROM orders WHERE customer_id = 10 AND order_date = DATE '2025-01-01';	CREATE INDEX orders_cust_date_idx ON orders(customer_id, order_date); 

SELECT * FROM orders WHERE customer_id = 10 AND order_date = DATE '2025-01-01';
Function Use on Indexed columns	TABLE ACCESS FULL;
FUNCTION in filter predicate	Using functions on indexed columns prevents the use of indexes.	Use a function-based index	CREATE INDEX emp_lastname_upper_idx ON employees(last_name);

SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';	CREATE INDEX emp_lastname_upper_idx ON employees(UPPER(last_name)); 

SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';

1)	How Index Works and Why It Speeds Up Queries?
An index is essentially a sorted structure that maps key column values to physical row locations.
When a query filters or joins on an indexed column, the database can:
•	Use the index to quickly locate relevant rows (INDEX RANGE SCAN)
•	Avoid scanning every row in the table (FULL TABLE SCAN)
This reduces I/O and CPU usage, especially on large tables, because only the subset of relevant rows is read.
2)	Guideline for index setup:
•	Columns in WHERE and JOIN condition clauses → high-priority index candidates
•	Highly selective columns first in composite indexes

3)	How to know if the newly added index helps or not?
Check the execution plan:
•	INDEX RANGE SCAN or INDEX UNIQUE SCAN → index is used
•	FULL TABLE SCAN → index is ignored
Other indicators:
•	Query runtime/cost decreases after adding the index
•	Number of logical reads drops
Notice that an index is not automatically helpful if:
•	Queries wrap the indexed column in a function (WHERE UPPER(indexed_column) = 'ALICE')
•	Column has very low cardinality (e.g., a boolean column which only has T and F two values)

4)	Shall I use Single vs. Composite Index?
•	Single-column index: best for queries filtering or sorting on one column.
•	Composite index (multi-column): effective when queries filter on multiple columns together. 
o	Example: WHERE a = ? AND b = ? → composite index on (a, b) is more efficient than two separate indexes.
o	Order matters: the first column should be with the highest cardinality or most commonly filtered, it would be the ideal column to create index on.
5)	INDEX SCAN variants and how to optimise?
Execution plans can show different scan types:
•	INDEX UNIQUE SCAN: single-row lookup using a unique or primary key index.
•	INDEX RANGE SCAN: range search on an ordered index.
•	INDEX FULL SCAN: scanning the entire index; sometimes faster than a table scan if the index is smaller.
•	INDEX FAST FULL SCAN: index-only scan for queries that do not require table columns.
6)	What to do if the execution plan ignores a new index?
•	Check statistics: stale statistics can mislead the optimizer
EXEC DBMS_STATS.GATHER_TABLE_STATS('schema','table');
EXEC DBMS_STATS.GATHER_INDEX_STATS('schema','index_name');
•	Check query expressions: functions, implicit conversions, or mismatched data types may prevent index usage.
•	Review selectivity: low-selectivity columns may not be used; consider filtering more selective columns first.
•	Use index hints cautiously:
Use SELECT /*+ INDEX(table index_name) */ ... 
Only if the optimizer consistently ignores a useful index.
•	Check plan after changes: always verify the execution plan after adding or modifying indexes to ensure they are actually being used.

### c)	Hints—Last Resort
SQL hints are directives embedded in a query that influence the optimizer’s execution plan. They don’t change the query result but can force specific behaviour and overwrite original explain plan when the optimizer’s default choice isn’t optimal.

1)	How does hint work?
Normally, the optimizer determines the most efficient execution plan based on available statistics and cost estimations. However, when statistics are incomplete, or the data distribution is skewed, the optimizer may choose a suboptimal path. 
Hints explicitly instruct the optimizer to use a specific plan component (e.g., index, parallelism etc.). Hints apply only to the statement where they appear, and their effect is visible in the EXPLAIN PLAN.

Index Hints
 
This tells the optimizer to use a specific index emp_dept_idx on the table e.
It is useful when the optimizer ignores an available index because of outdated statistics or complex conditions.
If emp_dept_idx is an index on column department_id, this hint forces an index range scan instead of a full table scan.

Parallelism Hints
Parallelism hints help because they tell the database optimizer to divide a large workload into smaller chunks and execute them simultaneously across multiple CPU cores or processes. It can significantly speed up large analytical or batch operations in this way.
Table-level parallelism:
 
PARALLEL(4) means using 4 parallel execution servers, it requests 4 parallel execution servers for the employees table
Each process scans a portion of the table concurrently, and results are merged.
Works best for large table scans and aggregation-heavy queries.
Statement-level parallelism:
 
This hint applies parallelism to the entire SQL operation, not just one table.
Parallel Index Scan:
 
This hint forces parallel index range scan using 8 parallel processes on index sales_idx.

When shall we use hint?
Hints should be a last resort, not a first choice.
They are useful when statistics are correct but the optimizer still makes a poor choice.
Avoid overusing hints—they make queries less adaptable to schema or data changes. 
DO NOT use PARALLEL on small tables—it can cause contention and excessive overhead.


### d) Incomplete Statistics
Even after you have fixed all the poor query, missing index etc., the explain plan still won’t show the improvement. This could be because the stale optimizer statistics of table could lead to faulty information and generate suboptimal execution plans.

1)	Most common cause

•	Tables or indexes were recently updated (INSERT/UPDATE/DELETE) but not analyzed.

•	‘DBMS_STATS’ has not been run for a long time.

2)	Solutions

•	Gather fresh statistics for affected tables and indexes

•	Check table for missing statistics
 
