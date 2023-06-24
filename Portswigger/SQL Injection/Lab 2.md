
### SQL injection vulnerability allowing login bypass : EASY

---

> Go to login page.

> Check that username field is vulnerable to SQL injection using `'`. Returns a server error, therefore it is vulnerable. 

> Login using administrator username and removing the password item from the query. Add any password in the password field.
```
administrator`--
```
> Successful login as admin, and can now change the admin email.

---
