Authentication is the process of verifying the identity of of a user or client. There are primarily three types of authentication.
- Something you __know__, such as a password or an answer to a security questions, these are _knowledge factors_.
- Some you __have__, a physical object like a phone or a token, these are _possession factors_.
- Some you __are__ or do, such as biometrics, these are called _inherence factors_.

> There is a key difference between authentication and authorization, authentication is the process of verifying that a user is who they claim to be, authorization involves verifying whether a user is allowed to perform a certain task.

Most authentication vulnerabilities arise from two paths
- Weak authentication mechanisms which fail to protect against brute force attacks
- Broken Authentication due to logic flaws in the implementation which can result in bypassing the the authentication mechanism entirely.

The impact of vulnerable authentication can be wide spread and expose further surface for more attacks, authentication vulnerabilities also exist in OAuth systems, which will be done in a different module.