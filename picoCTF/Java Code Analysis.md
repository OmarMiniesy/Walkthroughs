
---

I go about solving this challenge by first taking a look at the website to understand the features it has, take note of the endpoints, the parameters that are used, and if there are any security features that seem suspicious.
##### Step 1: Examining the Website

With BurpSuite Proxy HTTP History on, we open the URL of the website and authenticate with the provided credentials `user:user`, and see 2 very interesting things.

1. On authentication, the application responds with a JSON object:

![](./screenshots/java-0.png)

2. That payload is then used in the following request as an `Authentication` header to fetch the data for the user in the `GET /base/books` request.

![](./screenshots/java-1.png)

> Looking at this token, we see there are 3 parts to it that separated by a dot `.`, which means it is a JWT.

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJyb2xlIjoiRnJlZSIsImlzcyI6ImJvb2tzaGVsZiIsImV4cCI6MTcyNjAzMzQ4NywiaWF0IjoxNzI1NDI4Njg3LCJ1c2VySWQiOjEsImVtYWlsIjoidXNlciJ9.ZshXJVcP9UC7XskTL3Svz8k3O0HU0SdTS2YX_jLd93A
```

Burp has this useful feature that can decode JWT tokens by selecting each part of the 3 parts alone that are separated by dots:

- First segment: 

![](./screenshots/java-2.png)

- Second segment: This shows some interesting information about the `role` of the user, as well as the `userId`, and the `email`. If we can possibly manipulate the content to be some of admin or root user, we might be able to access the data of that super user.

![](./screenshots/java-3.png)

- Third segment: The third segment cannot be decoded, and after some research, it turns out that the third segment is the *signature* of the token. 

> The signature of the token is used to verify that the token is indeed the correct token, and has not been manipulated.

Looking at the website again, we see that once we login, there are 3 books, and there is a book called `flag`, but it is only accessibly by the admin user.

![](./screenshots/java-4.png)

If we open it, we are denied access:

![](./screenshots/java-5.png)

Now that we know that there is JWT security, and we need to escalate privileges from user to admin, lets take a look at the source code and see how it is managed, so we can identify a flaw that we can exploit to level up our user.

##### Step 2: Examining the Source Code

The first thing I opened was the `README.md` file to check out the documentation, and we stumble upon this:

1. 
![](./screenshots/java-6.png)

This details that in the `configs` folder, there are 2 users created, the `user` and the `admin`.

2. 
![](./screenshots/java-7.png)

We also see that there is a `security` folder that describes how the JWT tokens are handled.

> Checking out the `configs` folder:

- In the `BookShelfConfig.java` file, we see the source code that created the `admin` user:

![](./screenshots/java-8.png)

We see that the username is `Admin`, and the email is `admin`.
- These are 2 of the fields that are present in the `JWT` token that we grabbed from the user.

> Me from the future, notice how this is the second user that is created, so maybe we can assume that the userId field in the JWT token is 2, since it was 1 for the user we logged in with.

Now, we can check the `security` folder, and see how JWTs are handled.

- Opening the `/models/JwtUserInfo` file in the `security` folder, we see that it describes the 3 fields that are needed in the JWT token:

![](./screenshots/java-9.png)

- Opening the `JwtService` file, we see the constraints needed for a JWT token, as well as the name of the function that generates the secret that is used to create the signature for the token.

![](./screenshots/java-10.png)

- We see that the secret is generated using the `secretGenerator.getServerSecret()` function. Opening the function and the file:

![](./screenshots/java-11.png)

> The function that is used is the `getServerSecret()` function, and we see that it obtains the secret by reading the file called `server_secret.txt`. However, I couldn't find the file anywhere in the folder, so it calls the `generateRandomString()` function, which is hardcoded to return `1234`.

Now, we have gathered almost all of the information needed to create our own JWT token that will trick the server into authenticating as an admin user.

- Copy the JWT token from the request above to fetch the books, and place it in the [jwt.io](https://jwt.io/) website.

![](./screenshots/java-12.png)

- Place the data inside the fields that is needed to become the admin, such as the admin email: `admin`, userid:`2`, and role:`Admin`, and place the secret `1234` in the signature box.

> Copy the resulting JWT token, and place it in the request in the `Authorization`.

To check if what we did works, I tried searching for the book named the `flag`:

![](./screenshots/java-13.png)

Using the search functionality, we tried searching for the flag, and added the new JWT token from the `jwt.io` website after adding the required information, and wee see that we get the result `type:SUCCESS`, meaning we managed to authenticate as the user.
- Notice how the `id` of the book is `5`.

> however, we still don't get access to the book content to read the flag. So I went back to the `README.md` file to see if there is an endpoint that allows us to read the content of a file.

![](./screenshots/java-14.png)

Notice how there is an endpoint to access the book's pdf using the ID, which we have as `5`.
- So, we change the path in BurpSuite to access the book while keeping the token we added.

![](./screenshots/java-15.png)

We see that the PDF is returned, but we do not understand the content, so we can open the response in browser to view the file.

![](./screenshots/java-16.png)

We get the flag: 
```text
picoCTF{w34k_jwt_n0t_g00d_d7c2e335}
```

---
