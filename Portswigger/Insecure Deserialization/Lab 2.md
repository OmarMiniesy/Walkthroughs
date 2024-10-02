### Modifying serialized data types : PRACTITIONER

---

Given the credentials `wiener:peter`, we can log in in the `My Account` page with Burp Suite Proxy HTTP History tab open.

![](./screenshots/2-1.png)

Viewing the `session` cookie, we see that it is a serialized object that is base64 encoded.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"ujpfow5md115ykf8zzdhydmh4mhfexb4";}
```

> We see that it is an object called `User` with 2 attributes, a `username` and an `access_token`.

We can utilize the fact that PHP uses the loose comparator `==` operator.
- One of the tricky things that happens is that when a string with no numbers is compared to the integer `0`, the operation returns true.

Therefore, we can try to change the value of the `access_token` attribute to an integer with the value 0, which is compared with the access tokens in the server which are strings.
- Hopefully this works, and the server returns true, and logs us in.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";i:0;}
```
- Setting the access token to an integer with value 0.

After that we base64 encode it using Burp Suite Decoder tab:
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtpOjA7fQ==
```

Placing the value in the `session` cookie in the browser and then refreshing, we see the `admin panel` tab.

![](./screenshots/1-4.png)

Opening it and deleting the user `carlos`, we complete the lab.

---
