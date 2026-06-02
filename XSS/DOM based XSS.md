- Various sources and sinks can be used to exploit DOM based XSS, some common ones include
```
document.write()
```
- This writes to the DOM unsafely using user input and can be used to write the `<script></script>` into the websites code.
- Similarly in certain cases the DOM writes into elements such as `<select>`, thus one must escape out of these to inject code.
- Certain cases such as `element.innerHTML`, may not accept scripts and one needs to use other payloads such as `<img src=x onerror=alert(1)>`

DOM based XSS can also exist in third party libraries such as Angular JS and jQuery, for example in jQuery the `attr()` function changes the attributes of DOM elements and can be used to say change the `href` tag to redirect to a malicious link using a backlink.

A potential sink is the `$()` selector function in jQuery , a classic vulnerability in this is the usage of hash-change to create auto scrolling to certain parts of the web page
```
$(window).on('hashchange', function() {
	var element = $(location.hash)
	element[0].scrollIntoView();
}
```
One can use a external server to trigger a hash change as follows
```
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```
Which causes the alert to run.

Angular JS also posses certain classic DOM based XSS vulnerabilities, when an Angular application uses the `ng-app` attribute in HTML, Angular will execute Javascript inside double curly braces anywhere inside the HTML.

In some cases responses from the server might be ran through `eval()` functions, if the responses contain user controllable data it might be possible to execute code in these cases by escaping out of bounds.

Comments, or stored data may also be used to perform such XSS by injecting code into the page using Stored XSS.
