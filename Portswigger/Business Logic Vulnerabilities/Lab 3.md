
### High-level logic vulnerability : APPRENTICE

---

> Similar to [[Portswigger/Business Logic Vulnerabilities/Lab 1|Lab 1]], login via the given credentials `wiener:peter`.

![wiener-login](./screenshots/wiener-login.png)

> Then head to the required item `Lightweight "l33t" leather jacket`.

![jacket](./screenshots/jacker.png)


> With BURPSUITE PROXY intercept on, capture the request that is sent when pressing on `add to cart`.

![add-to-cart](./screenshots/add-to-cart.png)

> The `POST` request captured containing the details of the product.

![lab3-post](./screenshots/lab3-post.png)

> We can only play with the `quantity` body parameter.
> Placing in a very large number, or a string produces an error.

![lab3-err](./screenshots/lab3-err.png)

![lab3-err1](./screenshots/lab3-err1.png)

> However, placing a negative number works.

![lab3-neg](./screenshots/lab3-neg.png)

> Heading over to the `my cart` page.

![lab3-cart](./screenshots/lab3-cart.png)

> Placing my order gets an error, that the total price cannot be less than 0.

![lab3-nlt0](./screenshots/lab3-nlt0.png)

> We have `100$` in store credit, so we need the total price to be less than 100 but more than 0.
> Trying to change the quantity parameter to another number and sending the request doesn't change the cart, but it adds the number in the cart to the number in the quantity parameter.

> So if we change the `POST` request above with -5 to 6, the total cart quantity will not be 4, but will be 1.

![lab3-6](./screenshots/lab3-6.png)

![lab3-cart-1](./screenshots/lab3-cart-1.png)

> We now need to add another item in a negative quantity so that the price is less than 100 store credit that we have.

> Going to the home page and choosing another product.

![tree](./screenshots/tree.png)

> Capturing the `POST` request that is sent when this item is sent to cart is exactly the same as the `POST` for the jacket, but with `productId=2` parameter changed.
> Sending -30 of them to see the total price.

![lab3-30](./screenshots/lab3-30.png)

> The cart total is still 223$.

![cart](./screenshots/lab3-cart-2.png)

> Adding -6 more of the item to reach a suitable price.

![lab3-6-tree](./screenshots/lab3-6-tree.png)

> The cart price is now 0.32$, less than the 100, so we can place the order and complete the lab.

![lab3-cart-3](./screenshots/lab3-cart-3.png)

---
