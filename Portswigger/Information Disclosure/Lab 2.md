### Information disclosure on debug page : APPRENTICE

---

Skimming through the website and having BURPSUITE PROXY HTTP history on.
- See that in the home page, there are HTML comments.

![](./screenshots/lab2-comment.png)

```
/cgi-bin/phpinfo.php
```

Visiting this directory.

![](./screenshots/lab2-debug.png)

Looking for the `SECRET_KEY` environment variable by pressing `CTRL + F`.

![](./screenshots/lab2-env.png)

```
usd23dgkwk4nmv8giz4dikkrmej6ao3d
```

Pasting this value in the submit box completes the lab.

---
