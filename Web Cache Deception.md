# Web Caches
A web cache is another server which sits in between the origin server and the user. Say a user make a request for a certain resource, the request first goes to the cache, the cache checks whether it has it or not, when it doesn't this is know as a _cache miss_. The request is forwarded to the origin server which gives the resource, this may or may not be stored in the cache depending on the rules it uses to determine what to store.

If it is stored the next request in the same session might be returned by the cache server rather than the origin server, this is known as a _cache hit_.
##### Cache Keys
When a cache gets a HTTP request it must check whether it has the cached response for the particular request stored. The cache makes this decision by generating a _cache key_ specific to a request using various elements of the request. If an incoming request matches the cache key for a previous request then the cache returns the stored response.
##### Cache Rules
Cache rules determine what kind of data can be stored and for how long, this varies from cache server to cache server. Static content is usually cached since it does not change, dynamic content is usually not cached since it can contain sensitive data and also the user must get the latest response from the server. Cache deception attacks exploit how these rules are applied, the most common types of rules are
- __Static File Extension rules__: Rules which match and check the file extension of the requested resource such as `.js` for javascript and the likes.
- __Static Directory Rules__: Matching all URL paths starting with a certain prefix, `/static` or `/assets`.
- __File Name Rules__: For common file names which remain unchanged such as `robots.txt` and `favico.ico`

# Web Cache Deception Attack
The major steps while crafting a cache deception attack is as follows
1. Identify a target endpoint that returns a dynamic response containing sensitive information, focus on points with `GET, HEAD, OPTIONS`methods. Since requests which alter the state of the origin server are mostly not cached.
2. Identify a discrepancy in how the cache and the origin server parse the URL path.
3. Craft a malicious URL which results in the cache storing the dynamic sensitive content in the cache. When the victim accesses the URL, their response is stored in the cache. Using Burp, you can then send a request to the same URL to fetch the cached response containing the victim's data.
##### Detecting Cached Responses
Various response headers can indicate whether a response was cached or not
- `X-Cache: hit` - The response was served by the cache
- `X-Cache: miss` - The cache did not contain the the response, mostly the request is cached after this
- `X-Cache: dynamic`: The origin server dynamically generated the content. Generally this means the response is not suitable for caching.
- `X-Cache: refresh`: The cache was outdated and needed refreshing.
Sometimes a `Cache-Control` header is also present that indicates caching and time for caching etc.

If there is a large difference in the response time between two consecutive requests, it may indicate that the response has been cached.
# Exploiting static extension cache rules
There exist two major types of path mapping on current day servers, Traditional URL mapping. For example
`http://example.com/path/in/filesystem/resource.html`
- Here the `http://example.com` points to the server `/path/in/filesystem` represents the directory path on the server and `resource.html` is the name of the resource

The second type of URL mapping is RESTful URL mapping, consider `http://example.com/path/resource/param1/param2`, this is a endpoint based system. A common discrepancy can be as follows, suppose someone requests `http://example.com/user/123/apikey` now if i change it to `http://example.com/user/123/apikey/abc.js`, the server might abstract this and ignore the `.js` however this may result in the cache server storing the response allowing a attack to read the API Key of another user by giving the user a malicious link.

One can test for these by appending random stuff to the end of the url and seeing if the response changes, checking various file extensions to see which ones get cached and do not get cached.

Delimiter discrepancies can also be exploited, finding a delimiter character which is recognised as such by the origin server but not the cache can result in the server returning a response for a request for sensitive information and the cache caching it because of its rules.

##### Normalisation Discrepancies
Normalisation is the converting of representations of url paths into a standard format, the most common example being path traversal sequences such as `/aaa/..%2fprofile`. Discrepancies in how and whether the origin server and cache server normalise different URL components can create ways to perform web cache deception attacks. To perform such attacks it is important to be able to detect normalisation by either servers after one has found delimiter characters which work,

One can detect normalisation on the origin sever as follows
- Choose a non cacheable resource such as a profile page,or something which uses the POST method, and check for normalisation on it; for example by modifying `/profile` to `/aaa/..%2fprofile`. If the response matches the base response the server performs the necessary normalisation for the path traversal sequence.
- If not the server does not decode the slash and dot-segment

One can also detect normalisation on the cache server by adding a sequence on a cacheable resource as follows `/aaa/..%2fassets/js/abc.js`
- If the response is not cached then the cache server does not normalise the sequence and most likely uses the `/asset/` directory prefix to check on whether to cache or not
- If the response is cached then the cache server may have normalised the sequence to `/assets/js/abc.js`
we can also add the path traversal sequence after the main directory like this `/assets/..%2fjs/stockCheck.js`
- If the response is no longer cached the server decodes the sequence resulting in `/js/stockCheck.js`
- If the response is cached it may mean it has not decoded anything
Also confirm whether a directory rule is being used by requesting for something like `/assets/aaa` and see if it indicates caching.


Origin server normalisation can be exploited by something like: `/assets/..%2fprofile`.  The cache interprets it as the literal string and caches it based on its directory rules however the server returns the profile page. For cache server normalisation exploits make sure to encode everything to prevent issues with delimiter characters an cache servers usually decode everything.

For cache server normalisation exploits something like `/profile%2f%2e%2e%2fstatic` can be used, the cache server will view it as `/static` while the origin server will see the entire literal, however it will not return anything since it is gibberish, one will need to find a delimiter character used by the origin server but not the cache, use a brute force list for this, say such a character is `;` in some case. The exploit will look like, `/profile;%2f%2e%2e%2fprofile`

##### Using file name cache rules
Certain cache servers cache specific filenames such as `robots.txt` or `favico.ico` or other static files, by matching the complete filename. These can be exploited if the cache server normalises path sequences while the origin server does not. 