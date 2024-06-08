
> Analyzing the source code, we see that the query we can inject into gets displayed when the query parameter `debug` exists.
> We don't need to include the `debug` parameter for it work.

![](./screenshots/natas14_1.png)

> We see also that the password gets printed when the query returns at least 1 row.

> Noticing that the query uses `"`, double quotes not single quotes, then the payload we use should also use these quotes.
> Using the basic payload:

```
" OR "a"="a
```

> And placing it inside the username and password fields.

![](./screenshots/natas14-2.png)

> The equal sign gets url encoded, but it works without encoding as well.
> Sending this request, we see the password returned.

![](./screenshots/natas14_3.png)

> To see how our payload was injected, we can add the `debug` parameter as a query parameter and observe the response.

![](./screenshots/natas14_4.png)

> This query prints everything in the `users` table as all conditions are true.

```
natas15:TTkaI7AWG4iDERztBcEyKV7kRXH1EZRB
```

---
