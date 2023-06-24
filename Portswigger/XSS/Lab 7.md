
### DOM XSS in jQuery selector sink using a hashchange event : APPRENTICE

---

> View the jQuery script

![jqscript](./screenshots/jqscript.png)
> It goes to whatever is after the `#` in the URL.

> It is user controllable, so try adding after the URL
```
#<img src=1 onerror=print>
```
> And indeed we get the print function screen.

> To make this exploit delivered to the server, use the exploit server.
> Add an `iframe` tag, such that whenever the lab is loaded, the exploit appears. Add it in the exploit body.
```
<iframe src ="https://0a0b00b9043b4ccf8331690900400055.web-security-academy.net/#" onload="this.src+='<img src=1 onerror=print()>'"></iframe>
```
> We make the iframe src the lab, and after the # add to this src an img.
> This img contains an error in its src, and the onerror attribute is called with the print() function as the lab requires.

---
