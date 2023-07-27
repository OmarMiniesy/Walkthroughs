
### Exploiting XXE using external entities to retrieve files : APPRENTICE

---

> Viewing any item and then pressing on `check stock` while having BURPSUITE PROXY HTTP history on.

![](check-stock.png)

> Viewing the `POST` request sent via HTTP history.

![](lab1-post.png)

> We see there is XML being used.
> We can add a document that defines an external entity with the path attribute set to the `/etc/passwd` file we need.
> And then call this xml entity to view the results in the application response.

> Edit the `POST` request to add the entity.
```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" > ]>
```

> Then call the entity to view it.
```
&ext;
```

![](lab1-xml.png)

> Sending this request, we see the application responds with the required data in the `/etc/passwd` file and the lab is solved.

![](Portswigger/XXE%20Injection/screenshots/etc-passwd.png)

---
