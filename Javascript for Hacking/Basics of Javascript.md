###### Hexadecimal
- Hexadecimal is just numbers but in base 16, in javascript these can exist inside quotations, double or single and inside grave ticks, but hex escapes are not allowed, for example
```
function a(){}
\x61()
```
Does not work!
- Also when using hex escapes a capital `X`, does not work one must use `x`, otherwise it will be interpreted as a literal character rather than hex, `'\x61'` works but `\X61` doesn't.
###### Unicode
- Unicode works inside strings and also can be used to call functions, doing `\u0061()` correctly calls the function.
- Unicode can be written as `"\u"` or `\u{}`, you must specify four hexadecimal characters.
- Using `\u{}`, allows one to use unicode code points, and we can use the full unicode range of characters.
###### Octal
- Octal escapes use base 8 and can only be used in strings, represented using a backslash, if the number is outside octal range you just get the number returned.
```
'\141' //a
"\8" //outside range so 8 is returned
```
###### Eval and escapes
- One can use hex escapes in `eval` since it first decodes the string before running it.
- Similarly to use unicode inside eval one must double escape the backslash otherwise the unicode will be ignored due to decoding done by eval.
###### Strings
- Strings can be double quoted, single quoted or template, in addition to using octal, hex, unicode etc, one can also use single character escape sequences.
```
'\b'//backspace
'\f'//form feed
'\n'//new line
'\r'//carriage return
'\t'//tab
'\v'//vertical tab
'\0'//null
'\''//single quote
'\"'//double quote
'\\'//backslash
```
- Escaping a character not part of a escape sequence results in it being treated as a normal character `"\H\E\L\L\O // HELLO"`.
- Using backslash at the end of a line allows one to continue it to the second line.
- Template strings allow usage of newlines but quoted and double quoted strings don't, using `\` doesn't allow a `\n` to be present in the string.
- Template strings allow execution of arbitrary javascript execution with the `${}` placeholders, one can also thus nest template strings inside template strings resulting in some weird javascript.
- One can also use template strings to call functions like this
```
alert`1337` // calls alert with the argument 1337
```
- Thus if a function returns itself we can write some bizzare javascript
```
function x() {return x}
x```````````
```
- And this compiles!
###### Call and Apply
- Call is the property of every function that allows you to call it and change the "this" value of it.
- Apply is pretty similar to call, only it allows us to supply and array as the arguments.

Consider the following code
```
function x() {
	console.log(this.bar);
}

let foo = {bar: "baz"}
x.call(foo)
```
Call changes the this value of the function `x` to `foo` and thus we can call `this.bar` and get a valid output without any issues.

The first argument call takes is the assigned `this` value of the function and the following values are the arguments passed to the function if it requires so.
