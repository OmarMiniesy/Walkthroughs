### DOM XSS in document.write sink using source location.search inside a select element : PRACTITIONER

---

Checking the source code of the application, we see a script when we open any product.

![jstore](./screenshots/jstore.png)

This code reads the `storeId` from the URL query parameters.
- If it exists, then it writes to the DOM an extra `option` in the `select` element using `document.write` the value of `storeId`.

To test, lets add a query parameter called `storeId` and give it a random string.
```
/product?productId=1&storeId=minso
```

If we look at the source code, we see that our string was indeed written to the DOM.

![mins](./screenshots/minsy.png)

> Therefore, we have a *source* which is the URL query parameter `storeId` and a *sink*, which is `document.write`. There is taint flow, as there is a direct flow of data between these two, and hence, a possible DOM based vulnerability.

We then need to break out of the `select` element, and then trigger an alert using an `img` with `onerror`.
```
/product?productId=1&storeId=</select><img src=1 onerror=alert()>
```

![](./screenshots/lab4-1.png)

---
