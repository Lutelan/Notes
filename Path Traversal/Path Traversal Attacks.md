### Reading arbitrary files via path traversal
Consider a application which displays, say images on its page using the following url
```
https://insecure-website.com/loadImage?filename=218.png
```
Say the server uses the `/var/www/images/` directory to store images which it displays on the page.  With this information if their are no protections against path traversal, in that case an attacker may be able to retrieve arbitrary files by just going back to the root location as follows
```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```
and so on. On windows systems both `../` and `..\` work, and in both cases the attacker would be able to access basically every file on the server.

### Common obstacles to exploiting path traversal vulnerabilities
Some applications might strip path traversal sequences like `../`, however in this case if they allow absolute locating of files, one might be able to just use `/etc/passwd` to access the required file.

On the other hand if the removal of sequences such as `../` is not recursive, one could use something like `....//` to achieve the same effect, as after stripping we get the required sequence, for windows based servers, one might use `..../\` as well.

Other bypasses may include URL encoding characters such as `/` or even double URL encoding them, certain applications require absolute paths for accessing images such as `/var/www/images/1.jpg`, in this case one can just use the `../` sequence to get oneself to the root directory.

Some applications require the file name in the url to end with a `.jpg` or `.png` or whatever they like, one can get around this by using a null byte to prevent reading of the filename after the byte by internal systems as follows
```
filename=../../../etc/passwd%00.png
```
### Preventing path Traversal Attacks
The most effective way to prevent path traversal attacks is to completely avoid passing user input to system file API's, if one does so make sure to verify user input data, converting paths to canonical paths is also advisable
