
### Username enumeration via account lock : PRACTITIONER

---


> Head to the login page in the My Account page.

![login](./screenshots/login.png)

> Try a wrong username and password and capture the sent POST request via BURPSUITE PROXY and send it to REPEATER.

![wrong-creds](./screenshots/wrong-creds.png)


> The response looks like this for wrong usernames and passwords:


![lab5-error](./screenshots/lab2-error.png)

> We need to find the correct username that when logged to multiple times with an incorrect password, locks.
> To do that, we use the BURPSUITE intruder clusterbomb attack, and run each username 5 times with a wrong password.

1. Place the placeholders in the username and password parameters.

![lab5-placeholders](./screenshots/lab5-placeholders.png)

2. Set the username payload to be the copied [list](https://portswigger.net/web-security/authentication/auth-lab-usernames).

![lab5-user-payload](./screenshots/lab4-user-payload.png)

3. Set the passwords to null payloads, effectively just repeating the same username x number of times. Set the Generate payload setting to 5, so as to repeat each username 5 times.

![lab5-pass-payload](./screenshots/lab5-pass-payload.png)

> Start the attack, and notice for which username does the response length change after multiple incorrect login attempts.

![correct-username](./screenshots/lab5-correct-user.png)

> Notice that the size is different, opening the response, we see a different response than the normal.

![lab5-response](./screenshots/lab5-response.png)

> This means that the username `adserver` we found is the correct username that exists on the website.
> Now using a sniper attack to determine the password using this [list](https://portswigger.net/web-security/authentication/auth-lab-passwords).
> Set the username as `adserver` and put the placeholders on the password.

![lab5-password-sniper](./screenshots/lab5-pass-sniper.png)

> Setting the payload as a simple list and pasting the copied passwords from the online list.

![lab5-passwords](./screenshots/lab5-password-payload.png)

> Start the attack and look for the response with the 302 status code signifying we have logged in and moved to a different page.

> The attack finished, and there was no redirect. However, there were 2 mainly seen sizes, `3292` signifying that i need to wait, and `3240` signifying wrong username or password.
> There was only one password with a response of a different size, `3162` that had no error messsages.

![lab5-correct-password](./screenshots/lab5-correct-password.png)

> Trying this password `aaaaaa` with the username `adserver` to log in completes the lab.
> Must wait a minute because the requests locked the account.

---


