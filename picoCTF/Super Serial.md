
---

Opening the link and poking around the source code for the `index.php` file we find nothing, and there are no other files that are being fetched.
- So, we need to perform some reconnaissance.

> Opening the `robots.txt` file, we see something interesting.

![](./screenshots/super-1.png)

Trying to open the `admin.phps` file, we get a 404 not found error.
- However, the extension `.phps` is used to display `php` source code, where the browser will display the file as source code.

> Since we know that we have the `index.php` file, lets see if `index.phps` will open.

![](./screenshots/super-2.png)

We see that there 2 files being referenced, the `cookie.php` file and the `authentication.php` file.
- Opening their source code as well:

