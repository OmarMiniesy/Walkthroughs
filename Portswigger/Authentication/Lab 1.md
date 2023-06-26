
### Username enumeration via different responses : APPRENTICE

---

> Head to the login page in the My Account page.

![login](./screenshots/login.png)

> Try a wrong username and password and capture the sent POST request via BURPSUITE PROXY and send it to REPEATER.

![wrong-creds](./screenshots/wrong-creds.png)

> The response looks like this for wrong usernames and passwords:

![response](./screenshots/lab1-response.png)

> This means that the website checks first for usernames, and once they are correct, checks for the password.
> We can use the given username [list](https://portswigger.net/web-security/authentication/auth-lab-usernames) in BURPSUITE INTRUDER to test all candidate usernames. 
> Once we get a different response, we can move on to the passwords.

> Take the request captured to INTRUDER and place the placeholders in the username parameter, and choose a sniper attack.

![lab1-sniper-users](./screenshots/lab1-sniper-users.png)

> Go to the payloads, choose simple list, and paste the usernames found in the list.

![lab1-user-payload](./screenshots/lab1-user-payload.png)

> Start the attack and observe the different responses. This can be examined through different response sizes. 

![lab1-response-size.png](./screenshots/lab1-response-size.png)
> See that the username `an` has a different response size. 
> Observing the response, see that it is different, claiming wrong password.

![lab1-wrong-pass](./screenshots/lab1-wrong-pass.png)

> Now we can start testing the passwords with the the correct username.
> Take the passwords from this [list](https://portswigger.net/web-security/authentication/auth-lab-passwords)

> Go to INTRUDER with the same request, and change the username to `an` and put the placeholders on the password parameter.

![lab1-sniper-pass](./screenshots/lab1-sniper-pass.png)

> Set the payload as a simple list and paste the passwords from the list.

![lab1-pass-payload](./screenshots/lab1-pass-payload.png)

> Start the attack and observe the different response sizes and status codes.

![lab1-correct-pass](./screenshots/lab1-correct-pass.png)

> This shows that a completely different page is opened, possibly the page after signing in.
> Trying the password as `jordan`, the login works.

![complete](./screenshots/lab1-complete.png)

---
