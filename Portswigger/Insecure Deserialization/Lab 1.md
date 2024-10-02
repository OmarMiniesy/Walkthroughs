### Modifying serialized objects : APPRENTICE

---

Given the credentials `wiener:peter`, we can log in in the `My Account` page with Burp Suite Proxy HTTP History tab open.

![](./screenshots/1-1.png)

Following the instructions of the lab, we open the `session` cookie.
- Burp Suite automatically decodes it from base64, and we see that there is a serialized object.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

> See that it is an object called `User`, with 2 attributes, `username` and `admin`. The `admin` attribute is of type Boolean and it is set to `0`.

A possible exploit technique would be to modify the value of the `admin` attribute to be `1`.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

After that, we must then encode the new serialized object in base64 such that it is in a format that the server understands.
- Using Burp Suite Decoder tab.

![](./screenshots/1-2.png)

```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30=
```

Placing the new value of the object as the cookie and then sending the request again in Burp Suite Repeater.
- An easier option would be to place it in the browser cookie directly to view the response in the browser.

![](./screenshots/1-3.png)

> I did this using an extension called Cookie-Editor, but this isn't necessary. It can be done by going to developer options, then visiting the Storage tab in Firefox and placing the value in the `session` cookie.

Saving and refreshing the page, we see the `admin panel` tab is displayed.

![](./screenshots/1-4.png)

Opening the admin panel and deleting the user `carlos` completes the lab.

---
