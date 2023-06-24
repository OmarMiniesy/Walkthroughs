
### SQL injection vulnerability in WHERE clause allowing retrieval of hidden data : EASY

---


> Check the different categories and realize that there is a new category parameter in the URL.

> Try adding `'--` only after the value of the category: gets unreleased items only for that specific category and not all items in the shop. Vulnerable to SQL Injection.

> Try using OR Boolean attack
```
' OR 1=1-- 
```
> Displays all items of all categories whether released or not.

---
