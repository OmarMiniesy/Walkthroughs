
### DOM XSS in innerHTML sink using source location.search : APPRENTICE

---

> Realize that the search entry is placed within a span tag.
> No need to break out, we can simply write our payload inside of it.

> Introduce an error using the img src attribute and then put the onerror attribute that calls the alert function.
```
<img src=1 onerror=alert(1)>
```

> The alert pops up and the lab is complete.

---
