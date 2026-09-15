SSRF vulnerabilities is a type of attack which allows server side applications to make requests to unintended locations, which could be internal systems as well as outside systems for botnet based attacks.Basic SSRF attacks are ones which utilise user controlled parameters to query back end services, this could be modified by users and allow access to local sensitive systems which trust requests coming from the server itself.

Circumventing basic SSRF defences can be done in multiple ways:
 - Embedding credentials in URL before the host-name using the @ character
 - Using the # character to indicate url fragments
 - Utilising DNS names which you control and contain whitelisted domains in the name.
 - Using alternative representations of IP addresses such as `127.0.0.1 -> 2130706433`
 - Using a domain which resolves to said domain/IP
 - Obfuscating strings using URL encoding etc.
 - Changing protocols

Utilising endpoints which result in open redirections on a website along with SSRF can allow bypassing SSRF filters since the domain name is from the website itself.

Blind SSRF's can be harder to detect since we do not see any obvious difference in responses, out of ban techniques such as using SSRF's for connection to servers which one controls can prevent this.

One can use these to sweep for commonly known vulnerabilities on the website to detect the