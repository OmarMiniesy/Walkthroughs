
### Manipulating WebSocket messages to exploit vulnerabilities : APPRENTICE

---

> Heading to the Live Chat page while having BURPSUITE PROXY HTTP history on.

![](./screenshots/lab1-1.png)

> We see the headers that are in place that initiate a websocket connection.
> Therefore, we are now going to use the WebSockets History tab in BurpSuite while using the chat feature.

> Sending `Hello` in the chat.

![](./screenshots/lab1-2.png)

> We see the message sent to the server, and then in message number 8, we see the message sent back to us again.

![](./screenshots/lab1-3.png)

> This is used to display the message we sent on the screen.

![](./screenshots/lab1-4.png)

> The data is sent in a JSON format, so we can try to put our payload that calls the `alert()` function in the message.

```
<img src=1 onerror='alert(1)'>
```

> Sending that via the chat feature, nothing happens.
> We see that the angle brackets and qoutes are encoded.

![](./screenshots/lab1-5.png)

> So we try to intercept the message being sent using proxy, and changing it there.

![](./screenshots/lab1-6.png)

> Changing it to our payload.

![](./screenshots/lab1-7.png)

> Forwarding all the remaining requests, we see the alert pop up.

![](./screenshots/lab1-8.png)

##### Note

> I tried sending the normal payload.
```
<script>alert(1)</script>
```
> But the server replies with:
```
<script>alert(1)<\/script>
```
> I think that breaks out of the script, not making it work.

---
