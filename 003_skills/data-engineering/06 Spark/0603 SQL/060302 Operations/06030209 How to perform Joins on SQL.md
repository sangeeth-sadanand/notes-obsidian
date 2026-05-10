---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060302 Operations/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: How to perform Joins on SQL?
---
# How to perform Joins on SQL?


> [!Summary] Summary
> 
> 
> - **JOIN** is used to combine rows from two or more tables based on a related column between them. 
> 
> ```python
> non_buyers = customers.join(orders, on="id", how="left_anti")
> ```
> 
> |**SQL Type**|**PySpark how argument**|
> |---|---|
> |`INNER JOIN`|`"inner"`|
> |`LEFT OUTER JOIN`|`"left"` or `"left_outer"`|
> |`RIGHT OUTER JOIN`|`"right"` or `"right_outer"`|
> |`FULL OUTER JOIN`|`"full"`, `"outer"`, or `"full_outer"`|
> |`LEFT SEMI JOIN`|`"semi"` or `"left_semi"`|
> |`LEFT ANTI JOIN`|`"anti"` or `"left_anti"`|
> 


- **JOIN** is used to combine rows from two or more tables based on a related column between them. 

## 1. The Common Join Types

To illustrate these, let's imagine two tables: **Customers** and **Orders**.

### **INNER JOIN**
This is the most common join. It returns only the rows where there is a **match in both tables**. If a customer hasn't placed an order, they won't appear. If an order doesn't have a valid customer ID, it won't appear.

```sql
SELECT Customers.Name, Orders.OrderDate
FROM Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

```python
# If you don't specify 'how', it defaults to 'inner'
inner_df = customers.join(orders, on="id", how="inner")
inner_df.show()
```

### **LEFT (OUTER) JOIN**
This returns **all rows from the left table**, and the matched rows from the right table. If there is no match, the result will contain `NULL` for the columns of the right table.

- **Use case:** "Show me all customers and their orders, including those who haven't ordered anything yet."
    
```sql
SELECT Customers.Name, Orders.OrderID
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

```python
left_df = customers.join(orders, on="id", how="left") # or "left_outer"
left_df.show()
```

### **RIGHT (OUTER) JOIN**
The exact opposite of a Left Join. It returns **all rows from the right table**, and matched rows from the left.
- **Use case:** "Show me all orders and the customers associated with them, including orders that might not have a customer assigned."

### **FULL (OUTER) JOIN**
This returns all rows when there is a match in **either** the left or right table. It basically combines the results of both Left and Right joins.

```python
full_df = customers.join(orders, on="id", how="outer") # or "full"
full_df.show()
```

### **Semi and Anti Joins**
These are incredibly useful for filtering and often faster than standard joins.
- **Left Semi Join:** Returns rows from the left DF that **have** a match in the right (but doesn't actually add columns from the right).
- **Left Anti Join:** Returns rows from the left DF that **do NOT have** a match in the right.

```python
# Show me customers who have NEVER placed an order
non_buyers = customers.join(orders, on="id", how="left_anti")
```


## 2. Specialized Joins
### **CROSS JOIN**
This creates a **Cartesian Product**. It matches every single row from the first table with every single row from the second table. If Table A has 10 rows and Table B has 10 rows, you get 100 rows.
- **Use case:** Generating all possible combinations (e.g., every shirt size matched with every shirt color).
```sql
SELECT Sizes.SizeName, Colors.ColorName
FROM Sizes
CROSS JOIN Colors;
```

```python
cross_df = customers.crossJoin(orders)
```

### **SELF JOIN**
This is just a regular join, but the table is joined with **itself**. You must use aliases (temporary names) to distinguish the two "versions" of the table.
- **Use case:** An `Employees` table where one column is `ManagerID` (which points to the `EmployeeID` of someone else in the same table).

```df
SELECT E.EmployeeName AS Staff, M.EmployeeName AS Manager
FROM Employees E
INNER JOIN Employees M ON E.ManagerID = M.EmployeeID;
```

```python
df_aliased = customers.alias("c1").join(customers.alias("c2"), F.col("c1.id") == F.col("c2.id"))
```

## 3. Best Practices & Performance Tips
1. **Always use Aliases:** As queries get complex, writing `Customers.Name` and `Orders.Name` is tedious. Use `FROM Customers AS c JOIN Orders AS o` to keep it clean.
2. **Filter early:** If you only need orders from 2024, put that in your `WHERE` clause. Joining two massive tables before filtering is a great way to make your database admin cry.
3. **Check your Keys:** Joins are fastest when the columns you are joining on (`ON c.id = o.customer_id`) are **indexed**.
4. **The "On" vs. "Where" distinction:**
    - `ON` defines how the tables relate.
    - `WHERE` filters the result set _after_ the tables are joined.
