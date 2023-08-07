
### Stored XSS into HTML context with nothing encoded : APPRENTICE

---

> We need to look for something that gets sent in later HTTP responses when this website makes a request.
> We see that if we view a post, there are comments.

![](./screenshots/lab2-1.png)

> We try leaving a comment, and insert a payload to display the pop up.

```HTML
<script>alert(1)</script>
```

> Posting the comment, we see that every time we visit the same page, we get a pop up.

![](./screenshots/lab2-2.png)

> Hence, this is a stored XSS attack.

---
