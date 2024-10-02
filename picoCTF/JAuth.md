
---

Opening the website and keeping Burp Suite Proxy HTTP History tab open, we login with the provided credentials: `test:Test123!`.

![](./screenshots/jauth-1.png)

- Notice how there are 2 requests, number 72 and 73, the first a `POST` request to `/auth` where the user gets assigned the `token`. And the second `GET` request is to obtain the `/private` page using that `token`.

![](./screenshots/jauth-2.png)

Taking a closer look at the token, we realize its a `JWT` token as it has 2 `.` full stops:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdXRoIjoxNzI1ODcxOTM4NjE1LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChYMTE7IExpbnV4IHg4Nl82NDsgcnY6MTA5LjApIEdlY2tvLzIwMTAwMTAxIEZpcmVmb3gvMTE1LjAiLCJyb2xlIjoidXNlciIsImlhdCI6MTcyNTg3MTkzOX0.U63ohFYKzSTU9ictS-1F_OVz5ZTtSqKVXjGXd-BLobY
```

Taking it to [jwt.io](jwt.io), we can decode and understand its values:

![](./screenshots/jauth-3.png)

We see that it has multiple fields of information, but one important one is the `role` that is set to `user`, which we want to change to `admin`.
- However, the third segment of the JWT, the blue part, needs a secret token to be able to produce the signature. This signature is produced using the `alg:HS256` algorithm.

> One known attack to conduct against JWT tokens is to completely remove the signature and the algorithm used to produce the signature, and keep the extra full stop at the end, as there have to be 2 full stops in the token.

To be able to remove the algorithm, I went to a website called [token.dev](https://token.dev/):

![](./screenshots/jauth-4.png)

The algorithm is now set to none, and the role is set to admin.
- Copying the new JWT and trying it out to see if it logs us in as admin.
- Make sure there is an extra full stop at the end.

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNzI1ODcxOTM4NjE1LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChYMTE7IExpbnV4IHg4Nl82NDsgcnY6MTA5LjApIEdlY2tvLzIwMTAwMTAxIEZpcmVmb3gvMTE1LjAiLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3MjU4NzE5Mzl9.
```

> Going to the request that obtains the `/private` page that we want access to, and placing the modified JWT we made, we get access to the admin page.

![](./screenshots/jauth-5.png)

And the flag is produced:

```text
picoCTF{succ3ss_@u7h3nt1c@710n_72bf8bd5}
```

---
