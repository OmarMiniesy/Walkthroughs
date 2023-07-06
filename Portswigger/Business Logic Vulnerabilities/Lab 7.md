
### Weak isolation on dual-use endpoint : PRACTITIONER

---

> We need to access the `administrator` account and delete the user with username `carlos`.
> My credentials `wiener:peter`.

> Logging in as `wiener`.

![wiener-login](./screenshots/wiener-login.png)

> We are presented with the my account page.

![lab7-account](./screenshots/lab7-my-account.png)

> We can change the password of the user, changing it to `peter` again, and capturing the `POST` request sent via BURPSUITE INTERCEPT HTTP history.

![lab7-req](./screenshots/lab7-req.png)

> Sending that request we get the page saying that the password was changed successfuly.

![pass-changed](./screenshots/pass-changed.png)

> The `POST` request above has several body parameters.
> To login as `administrator`, we need to tamper with those parameters.

> First, changing the username to `administrator`.
> Second, try removing the `current-password` parameter with its value and see the response.

![lab7-req-new](./screenshots/lab7-new-req.png)

> The webpage responds normally, meaning the change we did worked.

![pass-changed](./screenshots/pass-changed.png)

> Trying to login with credentials `administrator:peter`.

![admin](./screenshots/admin.png)

> We get the admin page, and above is the admin panel.

![lab7-carlos](./screenshots/lab6-carlos.png)

> Accessing it, we can delete the `carlos` user to complete the lab.

---

