
### File path traversal, traversal sequences stripped non-recursively : PRACTITIONER

---

> Similar to [[Portswigger/Directory Traversal/Lab 1|Lab 1]], burpsuite filters images.
> To overcome this, go to PROXY HTTP history, and press on `Filter: Hiding CSS, image ...`

![show-all](./screenshots/show-all.png)
> And press on show all at the bottom.

> Refresh the lab page and see all `GET` requests with images.
> Choose one to play with and send to repeater.

![lab3-get-req](./screenshots/get-req.png)

> Change the value of the `filename` parameter. Since the Lab header says they are stripped non recursively, we will try the nested traversal sequences.
> Otherwise, i would've first tried using absolute path, then normal traversal sequences.

> Using nested traversal in the value of the `filename` parameter.
```
....//etc/passwd
```

![err-400](./screenshots/err-400.png)

> Keep adding nested upward jumps until it works.
> It works with 3 jumps.
```
....//....//....//etc/passwd
```

![etc-passwd](./screenshots/etc-passwd.png)

> This shows the `/etc/passwd` file, and the lab is complete.

---
