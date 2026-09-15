NoSQL injections similar to SQL injections, provide the ability to bypass authentication, retrieve data and enumerate information. For a start off, here is a simple introduction to NoSQL databases [[NoSQL Databases]] 

There exist 2 primary types of NoSQL injections, syntax injections which break query syntax allowing us to inject our own payloads, and operator injections where one uses NoSQL query operators to manipulate queries.

### Syntax Injections.
Detecting syntax injections comes down to testing and stretching the limits of the websites parsing and sanitisation capabilities, using fuzz strings as follows does exactly that,
```
'"`{ ;$Foo} $Foo \xYZ
```
In some applications one might need to fuzz hex characters or other things differently maybe doing something like `\u0000` or something else.

One can also determine which characters are being processed by utilising the character with and without backslashes to escape them out. One can confirm conditional behaviours as well using `&&` and `||` for OR's and AND's using strings like `' && 0 && 'x` and `' && 1 && 'x`

One can use these conditionals to override existing conditions using pipes and etc as well.
### Operator Injections.
NoSQL databases such as MongoDB utilise various query operators, the most common being
- `$where` - Matches documents that satisfy a JavaScript expression.
- `$ne` - Matches all values that are not equal to a specified value.
- `$in` - Matches all of the values specified in an array.
- `$regex` - Selects documents where values match a specified regular expression.

One can submit query operators inside JSON itself or even inside URL parameters, as follows
```
{"username":{"$ne":"invalid"}} OR username[$ne]=invalid
```
If that doesn't work either try:
1. Converting the request method
2. Changing the content type
3. Adding JSON to the message body

One can exfiltrate data using this with the help of conditionals, the most simple example being using the following payload to check parameters and values character by character.
```
admin' && this.password[0] == 'a' || 'a'=='b
```
One can also use this exact technique for identification of field names as follows
```
admin' && this.username!='
```
However there is a better method using operators which allows us to find the names character by character instead of guessing.

Checking for operator injections is simple using the `"$where":"0" and "$where":"1"`, if there is a difference in response then well operator injections are most likely possible.

One can also extract field names this way using the `.match()` function
```
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```
Exfiltrating data is also possible using this or using regex. One can also use timing based methods if their is no difference in the response itself using the `sleep` javascript function which can be used inside `$where` operators. Unions are easily implemented as just OR operations of numbers. Every single number $b < ( 1<< n)$ is a subset of $\{0,1,2,\cdots,n-1\}$.