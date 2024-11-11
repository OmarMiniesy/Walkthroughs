### Password reset broken logic : APPRENTICE

---

My credentials `wiener:peter`.
- Victim username `carlos`.

> Studying the forget password and the reset password functionality. First choose forget password.

![](Pasted-image-20230702213946.png)

Then enter our username `wiener` to reset our password.

![](Pasted-image-20230702214048.png)

After that visit the email client, and click the link that we received to reset our password.

![](Pasted-image-20230702214128.png)

Opening the link, and entering a new password `mins`.

![](Pasted-image-20230702214223.png)

> Having the BURPSUITE PROXY HTTP history tab open, there is a `POST` request containing a token `temp-forgot-password-token`, and the username for the user who forgot password, and the new password.

![](Pasted-image-20230702214428.png)

Submitting this request simply redirects us to the home page of the website. 
- So if we play in this request and get the same response and no errors, we might be moving in the correct direction.

> Assuming the website has no proper token validation, lets delete the values of the token, or simply insert anything, and see the response of the website.

![](Pasted-image-20230702214633.png)

Since it responds normally, lets try changing the user to carlos, and submitting the request.

![](Pasted-image-20230702214714.png)

Now, we have theoretically changed the password for the user with username carlos to mins.
- Trying to log in works, and the lab is solved.

![carlos-account](./screenshots/carlos-account.png)

---
