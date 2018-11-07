
# Grouping Data with SQL

## Introduction

In this section, you'll learn how to use aggregate functions in SQL.

## Objectives

You will be able to:

* Write queries with aggregate functions like `COUNT`, `MAX`, `MIN`, and `SUM`
* Create an alias for the return value of an aggregate function
* Use `GROUP BY` to sort the data sets returned by aggregate functions
* Compare aggregates using the `HAVING` clause


```python
import sqlite3
import pandas as pd
```

## Database Schema
<img src="Database-Schema.png">

## Connecting to the Database


```python
conn = sqlite3.Connection('data.sqlite')
cur = conn.cursor()
```

## Groupby and Aggregate Functions

Lets start by looking at some groupby statements to aggregate our data.


```python
#Here we join the offices and employees tables in order to count the number of employees per city.
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by city;""")
pd.DataFrame(cur.fetchall())
```

## Ordering and Aliasing
We can also alias our groupby by specifying the number of our selection order that we want to group by. Additionally, we can also order or limit our selection with the order by and limit clauses.


```python
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
pd.DataFrame(cur.fetchall())
```

## Retrieving Column Names
Recall that we can also retrieve our column names when using sqlite3 (note that this will be the default behavior in other environments such as sql workbench)


```python
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
df.head()
```

## Aliasing our Aggregate Function Name
Now that we can view our column names, we can also practice using alias's to name our aggregations.


```python
cur.execute("""select city,
                      count(employeeNumber) as employeeCount
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
df.head()
```

## Other Aggregations

Aside from count() some other useful aggregations include:
    * min()
    * max()
    * sum()
    * avg()


```python
cur.execute("""select customerName,
                      count(*) as number_purchases,
                      min(amount) as min_purchase,
                      max(amount) as max_purchase,
                      avg(amount) as avg_purchase,
                      sum(amount) as total_spent
                      from customers
                      join payments
                      using(customerNumber)
                      group by customerName
                      order by sum(amount) desc;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```


```python
df.tail()
```

## The having clause

Finally, we can also filter our aggregated views with the having clause. The having clause works like the where clause but is used to filter data selections on conditions post the group by. For example, if we wanted to filter based on a customer's last name, we would use the where clause. However, if we wanted to filter a list of city's with at least 5 customers, we would using the having clause; we would first groupby city and count the number of customers, and the having clause allows us to pass conditions on the result of this aggregation.


```python
cur.execute("""select city,
                      count(customerNumber) as number_customers
                      from customers
                      group by city
                      having count(customerNumber)>=5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

## Combining the where and having clause
We can also use the where and having clause in conjunction with each other for more complex rules.
For example, let's say we want a list of customers who have made at least 3 purchases of over 50K each.


```python
cur.execute("""select customerName,
                      count(amount) as number_purchases_over_50K
                      from customers
                      join payments
                      using(customerNumber)
                      where amount >= 50000
                      group by customerName 
                      having count(amount) >= 3
                      order by count(amount) desc;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```


```python
df.tail()
```

## Summary

After this section, you should have a good idea of how to use aggregate functions, aliases and the having clause to filter selections.
