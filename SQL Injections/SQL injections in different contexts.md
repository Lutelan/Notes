SQL injections can also be performed inside XML in web requests, for example something like this
```
<stockCheck>
	<productId>123</productId>
	<storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
We have encoded S to make sure simple checks don't work, if the server checks for SQL it is benefitial to just XML encode all the text.

