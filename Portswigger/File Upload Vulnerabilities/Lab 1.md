
### Remote code execution via web shell upload : APPRENTICE

---

> We need to read the contents of the file at `/home/carlos/secret`.
> Given login credentials `wiener:peter`.

> Logging in as wiener.

![](./screenshots/lab1-login.png)

> We see this my-account page.

![](./screenshots/lab1-account.png)

> We see there is a file upload in the `browse` button.
> Uploading an actual picture to monitor the websites behaviour while having BURPSUITE PROXY HTTP history on.


> Our account page now looks like this:

![](./screenshots/lab1-acc-img.png)

> Viewing this image in a new tab and observing the request.

![](./screenshots/lab1-img.png)

> We see that the images are stores in the `/files/avatars/` directory.
> So the browse button uploads files into this directory.

> So now if we upload a shell that reads the contents of the `/home/carlos/secret` file using the browse button.
```PHP
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
> File is called `shell.php`.

> Sending the `GET` request above to BURPSUITE REPEATER and changing the path to our `shell.php` file.

![](./screenshots/lab1-shell-req.png)

> Sending this request we get a response with the contents of the required file.

![](./screenshots/lab1-shell-res.png)

> Copying this value:
```
ORzhaY3dEOs0wB7Yq34PTeanRQ0Deg0j
```

> Submitting it in the submit solution box completes the lab.

---
