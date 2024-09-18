
---

Opening the website link, we see there is a green button with the text 'flag'.
- Clicking on it states that we are not `picobrowser`.

![](./screenshots/picobrowser-1.png)

Opening this request in Burp Suite Proxy's HTTP History tab, we see the request being sent, and the data that is output in the picture above is actually the value of the `User-Agent` header.

![](./screenshots/picobrowser-2.png)

Sending this request to repeater, and changing the value of the `User-Agent` header to `picobrowser`.

> Follow the redirection.

![](./screenshots/picobrowser-3.png)

We see the flag is output.

```text
picoCTF{p1c0_s3cr3t_ag3nt_e9b160d0}
```

---
