
### User role can be modified in user profile : APPRENTICE

---

> Given credentials `wiener:peter`.
> Can access the admin page at `/admin` only with `roleid=2`.

> Loggin in as `wiener`.

![](./screenshots/lab3-login.png)

> There are no cookies, hidden body parameters, or query strings that exist.
> Trying the change email feature to check for these data stores.

![](./screenshots/lab4-change-email.png)

> With BURPSUITE PROXY HTTP History on, we see a `POST` request is sent to change email.

![](./screenshots/lab4-post.png)

> We see that in the response, there is a key-value store with `"roleid"=1`.
> Sending this request to BURPSUITE REPEATER and adding in the JSON body parameter `"roleid":2` and sending it.

![](./screenshots/lab4-repeater.png)

> Refreshing the page in the browser, we see the admin panel button is now visible.

![](./screenshots/lab4-admin-pan.png)

> Opening it and deleting the user carlos completes the lab.

![](./screenshots/lab3-admin-pan.png)

---

