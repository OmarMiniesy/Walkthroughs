
### Referer-based access control : PRACTITIONER

---


> Admin credentials `administrator:admin`.
> Given credentials `wiener:peter`.

> Login as admin.

![](./screenshots/lab6-admin.png)

> View the admin panel and check out the feature.


![](./screenshots/lab6-panel.png)

> With BURPSUITE INTERCEPT HTTP history on, observe the requests that are sent.

![lab13-req](./screenshots/lab13-req.png)

> The `referer` header is used and its value is the `/admin` page.
> So if we try to access this page as wiener but leave the `referer` header, maybe we can access this page.

> To do that, login as wiener and capture the session cookie from the `GET /my-account?id=wiener` request.

![](./screenshots/lab13-wiener.png)

```
session=idiX4zayhwcdvRHFRDqVuabCdVrnc4MX
```

> Replacing the session cookie in the `POST` request with this one, and changing the `username` parameter to wiener to upgrade wiener.

![](./screenshots/lab13-exp.png)

> Sending the request works, and the lab is complete.

---
