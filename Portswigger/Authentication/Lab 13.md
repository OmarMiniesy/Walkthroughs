### Password reset poisoning via middleware : PRACTITIONER

---

> My credentials `wiener:peter`.
> Victim username `carlos`.

> What we want to do here is to get access to the reset password page for the account with username `carlos`.
> To do that, we need to get his reset-password token.

> Studying the reset password feature, it is similar to [[Portswigger/Authentication/Lab 12|Lab 12]].

![](Pasted-image-20230702213946.png)

> Then enter our username `wiener` to reset our password.

![](Pasted-image-20230702214048.png)

> After that visit the email client, and click the link that we recieved to reset our password.

![](Pasted-image-20230702214128.png)

> Opening the link, and entering a new password `mins`.

![](Pasted-image-20230702214223.png)

> This `wiener` account has a password reset token value of:

![](Pasted-image-20230702222720.png)


> Now, this works for a user we have access to their email account. 
> We need to make the user `carlos` send to us from his email account his token somehow.

> To do so, we take the first `POST` request that points to `/forgot-password`.

![](Pasted-image-20230702221943.png)

> This is the request that sends the email to the client. So instead of sending to `wiener`, lets send to carlos.
> Moreover, using the `X-Forwarded-Host` HTTP header, we can make the host that sends to `carlos` email our own exploit server.
> Therefore, we will be able to see the requests carlos makes.

![](Pasted-image-20230702222547.png)

> Now in our exploit server logs, we see a different IP for the user carlos, and a different token than the one for wiener.

```
temp-forgot-password-token=ru9Wxpx1H4lhBH1UhrLODrlEi9UaNTjA
```

> Now, what we need to do, is open the change password link for weiner, but make the token the one for carlos.
> Do that by changing the value in the URL.

![](Pasted-image-20230702223045.png)
> This has the old token, we can paste the one we have for carlos, and set the password to `minso`.

![](Pasted-image-20230702223130.png)


> Now if we login as carlos with password minso we complete the lab.

![carlos-account](./screenshots/carlos-account.png)

---
