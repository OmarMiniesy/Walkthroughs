
### Authentication bypass via flawed state machine : PRACTITIONER

---

> Given credentials `wiener:peter`.
> Goal is to access the admin interface and delete user with username `carlos`.

> Login via given creds.

![wiener-login](./screenshots/wiener-login.png)

> We are then asked to select a role. We are looking for admin, or something of the sort.

![role](./screenshots/role.png)

> The only available ones are:

![roles](./screenshots/roles.png)

> Choosing either one, but monitoring the request in BURPSUITE PROXY HTTP history.

![lab11-req](./screenshots/lab11-req.png)

> Changin the role to `admin` and resending the request.

![lab11-err](./screenshots/lab11-err.png)

> We can't do that, we must first login. Hence, there is a sequence that must be followed.
> Trying to alter the sequence of requests might work.

> With BURPSUITE PROXY on, login as `wiener`.
> Forwarding the `POST` request with the login creds.

![lab11-post-1](./screenshots/lab11-post-1.png)

> But then dropping the `GET` request right after responsible for selecting the role.

![lab11-get-1](./screenshots/lab11-get-1.png)

> We get an error. Opening the lab again, we are redirected to the home page of the admin user.

![lab11-admin-home](./screenshots/lab11-admin-home.png)

> Opening the admin panel to delete the carlos user completes the lab.


![lab11-carlos](./screenshots/lab6-carlos.png)

> Accessing it, we can delete the `carlos` user to complete the lab.

---