
### Inconsistent security controls : APPRENTICE

---

> Similar to [[Portswigger/Business Logic Vulnerabilities/Lab 5|Lab 5]], we need to delete `carlos`.
> Moreover, while registering, we need to use the email present in the email client.

![lab6-email](./screenshots/lab6-email.png)

```
@exploit-0a5100ed041ebe84814a295d0181003a.exploit-server.net
```

> Visiting the registration page.
> Noticing that admin privileges might be available if an account with the `dontwannacry` domain logs in.

![lab6-register](./screenshots/lab5-register.png)

> Registering with the email given in the client and using credentials `mins:mins`.

![lab6-reg](./screenshots/lab6-reg.png)

> After registering, checking the email client shows a link to complete registration.

![lab6-email-client](./screenshots/lab6-email-client.png)

> After completing the process, head to my account to login with the same credentials `mins:mins`.

![mins-login](./screenshots/mins-login.png)

> Logging in, we are presented with a page to update the email. Trying the `dontwannacry` domain to have admin privileges.

![lab6-newemail](./screenshots/lab6-newemail.png)

> After updating the email, we see the admin panel button appears, meaning we successfuly gained admin privileges.

![lab6-admin-page](./screenshots/lab6-admin-page.png)

> Pressing on the admin panel, we are presented with the list of users.

![lab6-carlos](./screenshots/lab6-carlos.png)

> Deleting carlos to complete the lab.

---
