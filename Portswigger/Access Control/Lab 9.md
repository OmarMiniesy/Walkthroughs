
### User ID controlled by request parameter with data leakage in redirect : APPRENTICE

---

> Given credentials `wiener:peter`.


> Logging in as wiener.

![](./screenshots/lab3-login.png)

> Notice in the URL a parameter `id=wiener`.

![][lab7-url.png]

> Changing the `id` parameter value to carlos.
> Monitoring the response using BURPSUITE INTERCEPT HTTP history.

![][lab9-redirect.png]

> We can see the account page for carlos, but viewing the response's source code, we see that we are actually on the `/login` page.

![][lab9-login.png]

> Hence, we were redirected to the `/login` page, but the data we needed was still output.
> Copying the API key to submit in the submit solution box.
```
VbCJXHqxj6hO8IWuNs9WrGFtXfreqnNh
```

![][lab9-submit.png]

> Completes the lab.

---
