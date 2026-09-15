Most websites handle concurrent requests using multiple threads, each of which read and write from the same database. Since, most code is not written with concurrency risks in mind, race conditions occur all over the place. Most common example of this being _limit-overrun attacks_. Such as redeeming a gift card multiple times, or re using a single CAPTCHA solution, or bypassing anti-bruteforce rate limits.

The true potential of race conditions can be realised from the statement, _everything is multistep_. To illustrate this, consider an example. Suppose a website assigns roles to users when the log in for the first time after registration on the basis of the domain of their email address. For certain business domains they are admins, for certain others customers and etc. Initially the user is in a _null_ state which when he logs in is changed. 

Since the admin role is probably the first one to be created in development, a vulnerability can occur where the website keeps to as admin by default until it is changed, and thus by forcibly navigating away one can retain these privileges. However if this is protected against, race conditions come into play. Removing the assumption that _requests are atomic_ from our mind, if a single `POST /login` request causes the user to change from admin to say another role. During the time it takes for the change to occur their is a window of time in which requests as an admin can be made. 

Thus every single HTTP step can have sub states inside websites which if timed right can be used to exploit them.

###### Single Packet Attack
Race time windows are often very small, and network jitter can make exploiting them incredibly tough. However the single packet attack method(only for HTTP/2) makes this easier. Since HTTP/2 allows being able to shove 2 HTTP requests inside a single TCP packet due too concurrent requests being allowed on single connections. To send multiple requests this way, one can do the following.

Broadly, this method involves sending bulk of the data of the requests inside a TCP packet and then only sending the completing request, this causes Nagle's algorithm to patch the requests into a single packet. Which eliminates close to all network jitter(but not server side jitter which is why we send 20-30 requests at a time). 

- If the request has no body, send headers but don't set the `END_STREAM` flag.
- If it has a body, send the headers and all the body data except the final byte.
- Wait like 100 ms to ensure the initial frames have been sent, disable `TCP_NODELAY`
- Send a ping packet to warm the connection and then send the with held frames. The single packet can be verifyed using wireshark.

> Quick note, this works for almost all servers, but doesn't work for static files on some servers, it results in the response being received before the request has been sent. This can be used as a way of testing whether a file is static or not.

##### Exploiting Methodology
This basic format for exploiting race conditions is pretty helpful. First one must predict potential collisions among requests, this could be multiple endpoints or a single one, somethings to consider when predicting collisions:
1. __How is the state stored__?: Data stored on persistent server side data structures is ideal for exploitation. Most entirely client side states can safely be skipped, user sessions can also be used to store user states, more on this later.
2. __Editing or Appending__?: We prefer to be editing data rather than appending it, exploits which append are most likely only vulnerable to limit overruns and nothing else.
3. __Do the requests affect the same record?__: If the requests effect the same records on the basis of say a session or token, it is better for exploitation purposes.

After predicting and selecting endpoints, one must probe for clues, sending requests with some difference, bench marking normal behaviour. Using the single packet attack, look for clues and deviations from benchmarked behaviour. Pay attention to response times, if a certain request has shorter response times, multiple threads being used could be a possibility. 

> Note that PHP locks on the session id by default, use separate sessions for each request to prevent them from being processes sequentially

Finally prove the concept, play around with timings, method's delays and etc to make sure to escalate the vulnerability, just because you couldn't exploit it, doesn't mean someone else could not. Be that someone else.

###### Some Tips
 - If a certain request just consistently is faster than other, you could try to induce client side delay, however this will prevent you from using the single packet attack method. Another way is to trigger rate limits by sending multiple dummy requests and thus being able to use the single packet attack viable even when delayed execution is required.
###### Future stuff
- Partial reconstruction attacks: Websites which create data structures in multiple states and one of the middle states is insecure can be vulnerable to partial reconstruction attacks.
- Single packet attack can be enhanced further by using TCP/TLS layer techniques to force maximum segment size up, or issuing packets out of order. 
- A more generic way to delay processing of specific requests in a single packet.
- Batching requests at the TLS layer rather than TCP.c
