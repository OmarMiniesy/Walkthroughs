
### Flawed enforcement of business rules : APPRENTICE

---

> Need to buy the item `Lightweight l33t leather jacket`.
> My credentials `wiener:peter`.


> Login via the given credentials `wiener:peter`.

![wiener-login](./screenshots/wiener-login.png)

> Then head to the required item `Lightweight "l33t" leather jacket`.

![jacket](./screenshots/jacker.png)

> And add it to cart.

![add-to-cart](./screenshots/add-to-cart.png)

> See that there is a coupon code present on the home page.

![lab12-coupon](./screenshots/lab12-coupon.png)

> Furthermore, signing in to the newsletter at the bottom of the home page.

![lab12-email](./screenshots/lab12-email.png)

> Gives another coupon code.

![lab12-new-coupon](./screenshots/lab12-new-coupon.png)

> We now have 2 coupons:

1. `NEWCUST5`.
2. `SIGNUP30`.

> Heading to the cart page.

![lab12-cart](./screenshots/lab12-cart.png)

> Applying the first coupon code.

![new-cust](./screenshots/lab12-newcuts.png)

> Results in:

![lab12-newcust-1](./screenshots/lab12-newcust-1.png)

> Trying to apply the same coupon again.

![new-cust](./screenshots/lab12-newcuts.png)

> Results in an error.

![lab12-again](./screenshots/lab12-again.png)

> Trying the second coupon.

![lab12-signup](./screenshots/lab12-signup.png)

> Results in:

![lab12-signup-1](./screenshots/lab12-signup-1.png)

> If we try `signup30` again we get the same error of repetittion.
> Trying to enter the first coupon, `newcuts5`, again, to see if the repetition check can be bypassed.

![new-cust](./screenshots/lab12-newcuts.png)

> Works, and the coupon is placed again.

![lab12-newcust-2](./screenshots/lab12-newcust-2.png)

> Trying to add `signup30` again works.
> Therefore, we keep on alternating the coupon codes until the price is within our limit of 100$.

![lab12-final](./screenshots/lab12-final.png)

> Now we can place the order to complete the lab.

---
