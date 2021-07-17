---
layout: post
categories: "en"
title: "SQL tips for fast big data queries"
date: 2021-07-12
---


## SQL tips for fast big data queries

Querying tables with SQL is the first part of any data science project, sometimes, it can be the slowest one too. To boost the performance of the extraction and loading, I’m writing down some tips to run queries fast.

I received these tips from my manager after she identified queries that took minutes to run. As a rule of thumb, if a query is taking over 30s, go back and find if there is any of the following mistakes in your SQL.

<br />

Common mistakes:
1. [SELECT * FROM table](#t1)
2. [Cartesian Joins](#t2)
3. [Joins on big tables](#t3)
4. [DISTINCT](#t4)
5. [WITH big tables](#t5)


<br />

---
### SELECT * FROM table <a name="t1"></a>

Today, data is stored in  big-data warehouses that use columnar-store tables, like Redshift, Hadoop and Big-Query. In this type of storage it is more efficient to access all rows from a column, than to access all columns for a single row. For this reason, avoid at all cost selecting all columns. Instead, choose only the fields that interest you.

⛔️ `SELECT * FROM table ;`

✅ `SELECT col1, col2 FROM table ;`



**Notes on column oriented and row oriented databases**:

Columnar-store is the opposite of more traditional row oriented data stores. In row-store, information is saved and retrieved one row at a time, they are easy to read and write, but it's not easy to aggregate or fetch all rows at the same time. This is the typical case of a "students database" in which we may want to see the data of a particular individual, like in many SQL tutorials.

Columnar oriented data store is typically used for OLAP (online analytical processing) applications. Here we would be mostly interested in performing calculations over fields, for example, fetching the amount of multiple transactions and calculating the mean.

Ultimately, in both situation a table looks exactly the same, but its performance is different depending on the types of query we ran against it. For row-oriented storage, we could easily see all the info for a given unit, and in column-oriented storage, we can process aggregates over columns very fast. [Further reading.](https://medium.com/bluecore-engineering/deciding-between-row-and-columnar-stores-why-we-chose-both-3a675dab4087#:~:text=In%20row%20oriented%20databases%2C%20these,data%20is%20read%20at%20once.)

<br />

### Cartesian Joins <a name="t2"></a>
Cartersian joins, aka cross joins, return all the possible combinations from the records of two tables, so if you are cross joining two tables with N rows, you'll get NxN results. Unless you are designing an experiment, or building a view or another table from scrach, you will never use this join. In some SQL editors, this is the default join when you don't specify otherwise, so always remember to write down either: `INNER JOIN` or `LEFT JOIN`. There's no need for any other, not even right joins.

⛔️ `SELECT t1.col1, t2.col2 FROM table1 as t1, table2 as t2 ;`

✅ `SELECT t1.col1, t2.col2 FROM table1 as t1 LEFT JOIN table2 as t2 ON t1.id = t2.t1_id ;`

<br />

### Joins on big tables <a name="t3"></a>

Talking about joins: don't join big tables with one another. It's better to do a two step process: first reduce each original table through a sub-query, then join these subqueries. Although it looks more complex to have nested queries, it will save time since the runtime will decrease significantly.


Instead of:

⛔️   
<pre>SELECT
  c.id,
  c.age,
  t.amt

FROM customers c                          
LEFT JOIN transactions t on c.id = t.cust_id
</pre>


✅
<pre>SELECT *    

FROM (
      SELECT id, age
      FROM customers
      ) c        
LEFT JOIN (
      SELECT cust_id, amt
      FROM transactions) t on c.id = t.cust_id
</pre>

### DISTINCT<a name="t4"></a>

The distinct clause is esentially a group by, but an expensive one. It is used to obtain unique information, usually when there are duplicates in your original table or in any of the joins. `DISTINCT` works by first sorting all the data, then grouping by all the fields included in the SELECT statement.

If you are only retrieving an index column, distinct is not a bad idea. But to use `DISTINCT` over multiple columns that are not indexes, is costly and could be avoided by other means: such as, use temporary tables with pre-aggregates, avoid certain joins.   


<br />

### WITH big tables<a name="t5"></a>

The with clause allows you to name a subquery so you can use it in a main query. This could be useful to structure your code, when you are nesting queries, but it can be expensive as it's running all your queries at once.  

Querying many full, big tables, at the same time is a bad idea. Sometimes when I have to create views or tables that involve many joins, like over 10, it's better to create intermediate temporary tables.

For ex, let's imagine we have a customers table, a transactions table, and a city table. We want to have the average amount of the maximum purchase made by a customer in each city.


Instead of running two queries at the same time:

⛔️   
<pre>
WITH my_big_table
  as (SELECT
      c.id,
      c.age,
      a.city,  
      max(t.amt) as max_amt      
      FROM customers c                          
      LEFT JOIN transactions t on c.id = t.cust_id
      LEFT JOIN cust_address a on c.id = a.cust_id
      group by 1,2,3
      )  

  SELECT a.city, avg(t.amt)
  group by 1 ;

</pre>

Create an intermediate temporary one, and do it two steps:


✅
<pre>
CREATE TEMPORARY TABLE AS max_amt_cust
  ( SELECT
      c.id,
      max(t.amt) as max_amt      
      FROM customers c                          
      LEFT JOIN transactions t on c.id = t.cust_id
      GROUP BY 1
  );

SELECT
  a.city(),
  max(m.amt) as max_amt      
FROM cust_address a
LEFT JOIN max_amt_cust m on m.id = a.cust_id
GROUP BY 1

</pre>
