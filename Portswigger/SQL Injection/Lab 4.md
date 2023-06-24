
### SQL injection UNION attack, determining the number of columns returned by the query : PRACTITIONER

---

> Open different categories and see that there is a category parameter.
> As stated, vulnerable to union injection.

> Try the exploit
```
' UNION SELECT NULL --
```
> Doenst work, so keep adding `NULL,` until it clicks.

> Works at 3 `NULL` added, so there are 3 columns in the query.
> Proof it works as the output wasnt just an error, but we saw other data outputted as well.

> Can try the `order by` technique. Once an error is produced then the number before that is the number of columns.
> As expected it is as 3, since the item has an id, name and price column.

---
