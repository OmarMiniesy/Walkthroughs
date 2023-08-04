
### DOM-based open redirection : PRACTITIONER

---

> We are looking for a script on the browser that changes the URL, or the `location` object.

> We see the `back to blog` button at the bottom of viewing a post.

![](./screenshots/lab1-1.png)

> Viewing its source code.

``` HTML
<a href="#" 
   onclick="returnUrl = /url=(https?:\/\/.+)/.exec(location); 
   location.href = returnUrl ? returnUrl[1] : &quot;/&quot;"
   >Back to Blog</a>
```

> Once we click this button, javascript code is executed through the `onclick` attribute . Furthermore, we see the `location.href` attribute being set by values present in the URL inside the `url` query paremeter.
> Therefore, we have a source, which is the URL that is user controllable, and we have a sink, which is the `location.href` attribute.
> This is a DOM based vulnerability.

> Analyzing the code, we see that it creates a variable called `return Url`.
> It uses regular expressions in the part `/url=(https?:\/\/.+)/` to match urls of the form `http` or `https` followed by `://` and any characters to form the URL after a query paremeter called `url`.
> It is basically searching for `url=https://<website>` in the URL.

> It then calls the `exec(location)` function, which retrieves the actual URL of the page, on the regular expression defined before.
> If there is a match, meaning that the regular expression defined abovce matches a part of the URL of the page, the matched part of the URL is stored in the `returnUrl` variable. `returnUrl[]` contains `url=https://<whatever>`.

> Finally, it sets the `location.href`, or the object that controls the redirection of the page, to either `returnUrl` or `returnUrl[1]` based on a ternary expression.
> If the `returnURL` variable has a match, then the `location.href` is set to `returnUrl[1]`. `returnUrl[1]` contains `https://<whatever>`.
> Otherwise, if there is no match, then the `location.href` is set to `"/"`, which is some sort of fallback URL.

> So, if we want to exploit this, we need to add the query parameter `url` by adding it after the `&` symbol.
> This parmeter will have value of `https://<any website>`, and this should redirect us to `https://<any website>`.

```
https://0a43002204e4431f816389e30091009c.web-security-academy.net/post?postId=9&url=<website>
```

> Setting `<website>` to be our exploit server.

```
https://https://0a43002204e4431f816389e30091009c.web-security-academy.net/post?postId=9&url=https://0a43002204e4431f816389e30091009c.exploit-server.net/
```

> If we enter this payload in the URL bar on top, we should be redirected to the exploit server, completing the lab.

---
