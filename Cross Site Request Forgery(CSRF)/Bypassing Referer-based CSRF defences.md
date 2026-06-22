The HTTP referer header is a header which is used when a user directs to a website by clicking on a link or submitting a form, it includes the origin from where the user has redirected to the new website. Some sites allow sensitive requests only when the request originates from the applications own domain however this can be bypassed in some ways.

__Validation depends on existence of header__
Some applications skip the validation of the header if it does not exist, the attacker must craft their csrf attack in such a way that the browser doesn't include the referer header when a request is being made, this can be done using a meta tag
```
<meta name="referrer" content="no-referrer">
```
Make sure to include this before the redirecting form or image tag only then will it be considered by the browser.

__Validation of Referer can be circumvented__
Certain applications check the correct referer header, using something like a subString command which only checks if the current websites origin, or some part of it exists in the referer header, rather than verifying the referer header further. Thus one must somehow change the url of the exploit page to contain the vulnerable website in it, so that it gets included in the referer header. This can be done using the `history.pushState(state, unused, url)` method. Leaving the first two empty, since `state` is used to specify the data for the state we want to push onto the history stack and `unused` is well not used. When this runs the `url` part is appended at the end of the current pages url, you can try this in a browser using the console. Thus running
```
history.pushState("", "", "/?vulnerable-site.net")
```
does this trick. In some cases this might not work as certain browsers strip the query string for security measures, this can be prevented using the following meta tag which sets the Referrer Policy.
```
<meta name="referrer" content="unsafe-url">
```
