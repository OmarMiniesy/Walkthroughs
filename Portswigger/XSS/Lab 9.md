
### Reflected DOM XSS : PRACTITIONER

---

> Open BURPSUITE Proxy and look at the HTTP history.
> See that there is another file containing the JS file script, called `searchResults.js`.

![eval](./screenshots/eval.png)

> The `eval()` function returns in the response a string.
> This response can be found in the HTTP history tab as one of the different files that come from the server.
> Choose the one whose `GET` request is for `/search-results`.

![response](./screenshots/response.png)

> This string can have a payload inserted that exits out of it, and includes the `alert()` function.
```
search=mins"-alert()
```
>Add the `"` character to break out the string, and the `-alert()` to add the alert() function.
>Add the `alert()` function after the `-` instead of the `+` because of URL encoding.

> This doesn't work, as we need to add an escape character before the `"` so as not to be read as double qoutes inside the string.
```
search=mins\"-alert()
```
> Now, we need to remove the right part containing the qouts and curly braces, and add our own.
```
search=mins\"-alert()}//
```

> Now the alert pops-up and we complete the lab.

---
