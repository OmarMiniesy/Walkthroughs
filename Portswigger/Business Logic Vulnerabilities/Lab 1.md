
### Excessive trust in client-side controls : APPRENTICE

---

Login via the given credentials `wiener:peter`.

![wiener-login](./screenshots/wiener-login.png)

Then head to the required item `Lightweight "l33t" leather jacket`.

![jacket](./screenshots/jacker.png)

> With BURPSUITE PROXY intercept on, capture the request that is sent when pressing on `add to cart`.

![add-to-cart](./screenshots/add-to-cart.png)

> The `POST` request captured containing the details of the product.

![post-cart](./screenshots/post-cart.png)

> See the `price` body parameter can be changed. 
> Changing the price to something trivial like 10 and then forwarding the requests.

![post-cart-price](./screenshots/post-cart-price.png)

> Visiting the cart page now shows that the item was added with the modified price.

![cart](./screenshots/cart.png)

> Placing the order completes the lab.

---
