
### URL-based access control can be circumvented : PRACTITIONER

---

> Access to `/admin` page is blocked.

![](./screenshots/lab5-denied.png)

> Using BURPSUITE PROXY HTTP history, send any request to REPEATER.

![](./screenshots/lab5-home.png)

> Adding the `X-Original-URL: /admin` header in the request and sending it.

![](./screenshots/lab5-pass.png)

> We see that we have access to the admin page.
> Viewing this response in browser.

![](./screenshots/lab5-browser.png)

> Pasting the link in browser opens the admin page.

![](./screenshots/lab3-admin-pan.png)

> Trying to delete the user carlos doesn't work.

![](./screenshots/lab5-denied.png)

> For the same reason as above.
> So instead, we delete the user from REPEATER.
> Checking the request to delete a user that didn't work to make changes.

![](./screenshots/lab5-req.png)

> Sending it to repeater and editing it.

![](./screenshots/lab5-delete.png)

> We add the delete page in the header at the bottom to access that page.
> We cannot add parameters to this header, so we add them at the top.

> Refreshing the lab page completes it.

---




