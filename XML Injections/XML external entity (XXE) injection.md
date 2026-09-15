XXE injections allow attackers to interfere with an applications processing of XML data. Lots of applications utilise XML to transmit data between the browser and the server. Applications which do this use standard libraries and APIs which allow dangerous functions. Standard parsers allow these even if they are not used by applications. [[XML Entities Crash Course]]
##### Retrieving Files using XXE
Defining custom entities as follows and including them in tags which return visible responses can cause file retrieval if there are no protections against XXE
```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> 
<stockCheck><productId>&xxe;</productId></stockCheck>
```
If multiple data values may exist where one can inject the reference to the custom entity reference, thus one must check them all out and see which one is visible in the response.

__SSRF using XXE__
Defining custom entities for URLs can result in SSRF attacks which allow us to touch internal systems and exploit SSRF, as follows
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
```
##### [[Blind XXE Vulnerabilities]]

### XInclude Attacks
Certain applications embed client side data into XML documents in the back end and not in the front end, which are then parsed accordingly. One example of this might be the usage of SOAP requests. A server might create a XML document using client input which is sent over to a SOAP server in the back end to obtain a response in XML itself.

XInclude provides the ability to create XML documents as fragments of sub documents. XInclude can be placed inside any data value in the XML document, and thus can be used as way to perform XXE attacks. To allow usage of XInclude one must include its namespace, the most common example is as follows to retrieve a file.
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```
The `parse` parameter defines the media type using which the document shall be parsed, by default being set to `applications/xml`.

### XXE via file uploads
Websites which allow file uploads for profile pictures or other things, might support the uploading of svg files which are nothing but an application of XML files themselves, thus if file uploads are not checked correctly uploading a malicious svg file can result in retrieval of arbitrary files as information in the SVGs. A very simple payload might be something like this
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg version="1.1"
     width="300" height="200"
     xmlns="http://www.w3.org/2000/svg">

  <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

### XXE attacks via modified content type.
HTML forms usually use the content type, `application/x-www-form-urlencoded`, changing this to one of the media types which allow parsing of XML input such as `text/xml`, might allow you to use XML based input without causing any particular difference in applications behaviour besides being able to exploit XXE.