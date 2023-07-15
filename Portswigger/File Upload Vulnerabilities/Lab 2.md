
### Web shell upload via Content-Type restriction bypass : APPRENTICE

---

> We need to read contents of the `/home/carlos/secret` file.
> Given login credentials `wiener:peter`.


> Logging in as wiener.

![](./screenshots/lab1-login.png)

> We see this my-account page.

![](./screenshots/lab1-account.png)

> We see there is a file upload in the `browse` button.
> Trying to upload a php shell to obtain the secret file, while having BURPSUITE PROXY HTTP history open
```PHP
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
> `shell.php`.

> We get this response.

![](./screenshots/lab2-repeater.png)

> We see that we cannot upload any files that are not of content type: `image/jpeg` or `/png`.
> Sending this `POST` request to BURPSUITE REPEATER and modifying the content type in the request.

![](./screenshots/lab2-modified.png)

> We modified the content type in the request in line 21, and we see that it is accepted.

> Now, we need to navigate to where the shell is on the website.
> If we go to the my-account page and click on where the image should be, we are supposed to open the php shell.

![](./screenshots/lab2-acc.png)

> Right clicking on the image and opening in new tab.

![](./screenshots/lab2-secret.png)

> Displays the secret, copying it:
```
JFcv88hrJ1MWrogWt7rTPox3FGd2kgAd
```

> Placing it in the submit box to complete the lab.

> We couldve went to where the script is using the same technique as [[Portswigger/File Upload Vulnerabilities/Lab 1|Lab 1]].
> Upload an actual image, capture the `GET` request via BURPSUITE by removing the filters, see the path of the image, and change the filename to `shell.php`. 

---
