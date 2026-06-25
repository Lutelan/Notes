### Keeping Users Logged In
A common feature that websites provide is the ability to stay logged into the page even when the browser sessions has been closed, so that one does not need to log back in when accessing the web page after time. The most common way in which websites do this is by using some sort of a _stay-logged-in_ cookie. 

If the generation of the cookie is not completely random and utilises, say the username and password or other details of the user in a unsafe manner, it could be possible to brute force the login cookie and access the user's account by creating a valid cookie. Even one way hash functions when not salted can be broken if the password used is a commonly known one. The cookie generation process could be analysed by the attacker if they can create their own accounts and see what usernames and passwords lead to which types of cookies.

Even if the attacker can't create accounts, he could steal people's cookies using other methods, such as XSS attacks and analyse how they are created. Even if the hashing algorithm is one way, using a weak password might result in the hash being identified.(Use salt's while encrypting).

### Resetting User Passwords
Since user's inevitably forget passwords, most websites provide a way to reset passwords. As the usual password based authentication is not possible the websites have to rely on other methods to make sure the passwords reset mechanism is only used by the correct user.

__Sending Passwords by Email__
If passwords are handled securely by the server it should be impossible for the sever to even send a plaintext password to the user by email. In this case the website might send a new temporary generated passwords via email. This is not the most secure since email is not meant for security and it also relies on the user changing this password. Many users also sync mails across devices which may lead to more attack surface.

__Resetting Passwords using a URL__
A robust method of resetting passwords is generating a unique url for the user to visit and thus reset the password. However if the URL generation is weak such as the following
```
http://vulnerable-website.com/reset-password?user=victim-user
```
Then it may be possible for the attacker to perform a reset request and then visit the url to reset the password. 

Also even if the token/cookie used is correct, the verification system on the back end to check which user's password to reset should be bullet proof. Some website check the token on visiting the reset website but forget to check it on form submission which could enable a attacker to change arbitrary users passwords.

If the reset link is generated dynamically using say the `Host` header or the `X-Forwarded-Host` header, the attacker could modify these and the user identifier to change the password reset link to a different domain allowing him to steal the uses reset token.

### Changing User's Passwords
Systems which allow a user to change his password usually require the user to enter his current password, if not implemented correctly then this might result in the ability to enumerate passwords using this functionality. If the server returns different responses on giving the correct current password or the wrong one. Using this one might be able to brute force the password.