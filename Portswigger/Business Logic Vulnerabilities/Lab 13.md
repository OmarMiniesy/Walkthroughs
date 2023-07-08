
### Infinite money logic flaw : PRACTITIONER

---

> My credentials `wiener:peter`.
> Need to buy the item `Lightweight l33t leather jacket`.


> Login via the given credentials `wiener:peter`.

![wiener-login](./screenshots/wiener-login.png)

> Signing in to the newsletter at the bottom of the home page.

![lab12-email](./screenshots/lab12-email.png)

> Gives coupon code.

![lab12-new-coupon](./screenshots/lab12-new-coupon.png)

> Moreover, there is a `gift card` item in the home page.

![gift-card](./screenshots/gift-card.png)

> Viewing its details and adding to cart.

![add-to-cart](./screenshots/add-to-cart.png)

> Heading to the cart page and placing the coupon code `SIGNUP30`.

![lab13-cart](./screenshots/lab13-cart.png)

> Results in:

![lab13-cart-1](./screenshots/lab13-cart-1.png)

> Placing the order.

![lab13-conf](./screenshots/lab13-conf.png)

> Our store credit is now 93.
> And we have a gift card code `g57dFPvCHA`.

> We can apply this gift card in the my account page.

![lab13-myacc](./screenshots/lab13-my-acc.png)

> Redeeming the code, we see our store credit is now 103 in total.

![lab13-conf-1](./screenshots/lab13-conf-1.png)

> Therefore, we can keep repeating this process to increase our store credit to be able to purchase the item `Lightweight l33t leather jacket`.

![jacket](./screenshots/jacker.png)

> Taking note of the HTTP requests that are important using BURPSUITE PROXY HTTP history.

![lab13-history](./screenshots/lab13-http.png)

1. `POST /cart` : Adding the gift card to cart.
2. `POST /cart/coupon` : Adding the coupon code in checkout.
3. `POST /cart/checkout`: Placing the order.
4. `GET /cart/order-confirmation` : Obtaining the gift-card code.
5. `POST /gift-card` : Redeeming the gift card code. 

> Since we need to automate these 5 requests, we are going to use a macro.
> This macro will work alongside BURPSUITE INTRUDER in each attack to perform these requests.

> Creating the macro.


1. Go to settings and then Sessions.

2. Click on Add to create a new rule, and then go to Scope and choose Include all URLs.

![img](./screenshots/Pasted-image-20230702004207.png)

3. Then go to Details and press on Add and choose Run a Macro.

![](./screenshots/Pasted-image-20230702004424.png)

4. Press Add again, and then add the requests in order to create the macro.

![lab13-macro](./screenshots/lab13-macro.png)

5. To use the correct gift card code each time, we go to the `GET /cart/order-confirmation?` request and press on configure items.

![lab13-config](./screenshots/lab13-config.png)

6. Then we press on add a parameter at the bottom to include our gift card code.

![lab13-add](./screenshots/lab13-add.png)

7. We then name it `gift-card` for simplicity, and to get its value, select it in the response at the bottom.

![lab13-par](./screenshots/lab13-par.png)

8. To take this custom parameter from this request and carry it to the next one to redeem it, we need to configure the `POST /gift-card` request.

![lab13-conf-2](./screenshots/lab13-conf-2.png)

9. Specifying that the `gift-card` parameter should be from the previous response.

![lab13-der](./screenshots/lab13-der.png)

> To test that the macro works, press on test macro and check that the gift-card in the last response is the same as that in the previous response.
> Also make sure that all requests are either 200 or 302 resp codes.

> Now that we created the macro, we use BURPSUITE ITNRUDER to generate indefinite requests, and with each request the macro is run, until we have enough store credit to buy the required item.
> Sending a dummy request to intruder and adding empty payload for a sniper attack.

![lab13-int](./screenshots/lab13-int.png)

> Choosing the payloads as null payloads and making it indefinite.

![lab13-payload](./screenshots/lab13-payload.png)

> We also need to make sure that these requests are carried out in order, so we make the max concurrent requests equal to 1.

![lab13-max](./screenshots/lab13-max.png)

> Finally, we run the attack and keep refreshing the my-account page to observe the store credit.
> Once i have enough credit, go to the required item and add it to cart.

![jacket](./screenshots/jacker.png)

![add-to-cart](./screenshots/add-to-cart.png)

![lab13-final](./screenshots/lab13-final.png)

> Placing order completes the lab.

---
