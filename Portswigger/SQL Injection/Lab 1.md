
### SQL injection vulnerability in WHERE clause allowing retrieval of hidden data : APPRENTICE

---


Check the different categories and realize that there is a new `category` parameter in the URL.

![](./screenshots/lab1-1.png)

![](./screenshots/lab1-2.png)


> Testing this parameter, we can try an injection character and see if the application responds in a weird way.

If we add the `'--` payload after the `category=accessories` query parameter, we see more items being returned.

![](./screenshots/lab1-3.png)

> Therefore, it is vulnerable to SQL injection.

Given this query
```SQL
SELECT * FROM table WHERE category=accessory AND released=1;
```

The injection `'--` we added removes the `AND` portion of the query, and so it returns all items in the table regardless of the `released` column.

> Now, we can try using a more complicated attack to list all items in the table, regardless of the `category`.

We can do this by injecting the `OR` statement, and adding a true condition afterwards, such that the query always evaluates to true:
```SQL
' OR 1=1-- 
```

This displays all items of all categories whether released or not.

---
