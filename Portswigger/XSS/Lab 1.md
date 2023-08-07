
### Reflected XSS into HTML context with nothing encoded : APPRENTICE

---


> Realize that there is an injection point in the search bar, because once i submit my search, it is displayed as data in the immediate response in the browser.

![xss](./screenshots/mins.png)

> Try adding this payload into the search bar.
> See also that it is placed as a query parameter in the URL.
```
<script>alert(1)</script>
```
> And the website responds with a pop-up window, informing us that it is vulnerable to reflected XSS.

![](./screenshots/lab1-1.png)

---
