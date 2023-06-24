
### SQL injection attack, querying the database type and version on MySQL and Microsoft : PRACTITIONER

---

> First get the number of columns in the output query using the normal test.
```
' UNION SELECT NULL, NULL --
```
> But this didn't work, and I expected only 2 or 3 columns evident by the data title, ID, and probably description.

> Since this is on MySQL and Microsoft, the comments in the end need an extra space, but trailing spaces are removed, so a dummy character is placed afterwards.
```
' UNION SELECT NULL, NULL -- -
```
> And it works, therefore, there are only 2 columns in the output query.

> Then, to output the database version, we know it is of type string. Therefore, we need to check which columns are of type string.
```
' UNION SECECT 'a', NULL-- -
' UNION SELECT NULL, 'a' -- -
```
> They both work, hence both of these columns are text type string.

> To output the database version, use the payload:
```
' UNION SECECT @@version, NULL-- -
```
> And the version is output as expected in a new row.

![DatabaseVersion](./screenshots/version.png)


---
