
### Insufficient workflow validation

---

> Given credentials : `wiener:peter`.
> Need to by the `Lightweight l33t leather jacket`.

> Login via the given credentials `wiener:peter`.

![wiener-login](./screenshots/wiener-login.png)

> We have 100$ in store credit, so lets try buying an item we can afford to observe the payment and checkout processes.

![sign](./screenshots/sign.png)

> Viewing this items details and then adding to cart.

![add-to-cart](./screenshots/add-to-cart.png)

> Going to cart and placing the order.

![lab10-cart](./screenshots/lab10-cart.png)

> With BURPSUITE INTERCEPT HTTP history, observe the sequence of requests sent after placing the order.

![lab10-history](./screenshots/lab10-history.png)

> The `GET` request is the request that confirms the order only for the item we added.
> If we add another item in the basket, and then resend that exact same `GET` request again, maybe we can break the logic.

> Adding the required `Lightweight l33t leather jacket`.

![jacket](./screenshots/jacker.png)

![add-to-cart](./screenshots/add-to-cart.png)

> Then resending the same `GET` request as above and observing the response.

![lab10-complete](./screenshots/lab10-complete.png)

> We see that we deducted only 64 dollars from store credit, the price of the old item, but the item we bough is for 1337$.

> We broke the sequence of requests by adding an item in the basket then resending the confirmation for a different item.

> The lab is complete.

---
