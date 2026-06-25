### Brute Force Attacks
Brute force attacks are attacks in which an attacker uses a system of trial and error to guess valid user credentials, however unlike the name suggests it doesn't always involve random guessing using wordlists, wordlists can be tuned for specific people and clues and contexts help make brute force attacks much more efficient.

- Brute forcing usernames is easy in situations where the username follows a set format such as `firstname.lastname@email.com`, some applications also disclose usernames such as in profile pages, HTTP responses also contain usernames and email addresses in certain cases.
- Password brute forcing is made more efficient by taking context clues along regarding the person, such as birthdays, names, etc.

__Username Enumeration__
Username enumeration is mostly enabled by difference in requests on the basis of whether the queried username exists or not, regardless of the password being correct or not, the differences are usually as follows
- __Status Codes__: Most responses will return the same status code, since most attempts will be completely wrong, thus one different status code can suggest that the username was correct.
- __Error messages__: In certain cases different error messages are displayed on the basis of whether only the username is correct or when both the username and password are wrong.
- __Response Times__: Variability in response times can indicate if the username is correct or wrong since if the username is correct checking the password could add substantial time to the response, especially if the password queried is quite long.

In certain cases IP blocking is used by websites to prevent brute force attacks and setting a timeout, this can be bypassed if the implementation is flawed using the `X-Forwarded-For:` header, which can trick the website into thinking that the IP address has changed, in other cases use of rotating proxies may be required.

### Flawed Brute force protection
Since a brute force attack likely involves many wrong guesses, thus brute-force protection involves slowing the attacker down in a a myriad of ways. One of the most common ways is __IP blocking__ the attacker in a timed way to slow them down. However in certain applications this counter for requests made can be reset if the IP logs in successfully.

So all the attacker needs to do is just slip in correct credentials to a account they own in between the brute force requests from time to time, to reset the counter and continue the brute force attack.

__Account Locking__
Account locking is a protection in which the user's account is locked from accepting login requests for a certain time period if too many login requests are used, however this may be used as a endpoint for username enumeration since it can identify if the username is actually correct.

Account locking is also not a complete protection since, using credential stuffing or password spraying bypass this protection.

__User Rate Limiting__
Rate limiting is a type of protection which results in your IP being blocked, if you make too many requests in a certain period of time. The IP can usually be unblocked in only the following ways
- Automatically after a certain period of time has elapsed
- Manually by the administrator
- Manually by the user after completing a CAPTCHA.

This method is sometimes preffered to account locking since it can prevent username enumeration, but it also has its flaws if implemented in an unsafe manner, also since it depends on the number of requests you send, it is possible to bypass the defence if you can work out how to guess multiple passwords in a single request.

> In some situations if the website uses JSON in the requests to send usernames and passwords, passing an array for the password or the username can allow one to input multiple creds in a single request.

### HTTP Basic Authentication
In HTTP basic authentication, the client receives a authentication token from a server, which is the base64 digest of the username and password concatenated together, this is included in the Authorization header by the browser every time the user decides to login or visit the website.

This is usually considered insecure, unless the website implements HSTS, the creds can be caught using a MiTM attack, it is also vulnerable to CSRF, it is mostly used for uninteresting pages, however even access to these could give the attacker a larger attack surface.