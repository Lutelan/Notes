Multi factor authentication is exactly what it suggests, usage of multiple factors to verify the identity of a user, most commonly in the form of a temporary verification code on a out of band device such as a mobile phone or a verification code sent to the email of the user.

Multi factor authentication is obviously better than single factor authentication, however similar to any other form of protection if not implemented correctly can be bypassed in a easy manner. The full benefits of Multi factor are achieved only when we verify multiple __different__ factors. Verifying the very email the user uses to login is not a good way to implement Multi factor authentication.

### Two factor authentication tokens
Verification codes are usually given to a user by some kind of a physical device, high security platforms provide a user with dedicated devices for such a purpose such as a RSA token or a keypad device. Applications also use verified applications such as Google Authenticator for this purpose as well.

SMS based verification is also used by some web applications, however it is much more open to abuse. There is potential for the SMS to be intercepted, also SIM swapping or cloning is a risk which would result in every message being sent to the user to be also sent to the attacker's phone with the sim.

> In certain extremely poorly designed applications, the website doesn't even check if you have completed the 2FA verification step and already puts you in the login state, just going back to the home page in this situation can give you access to the account.

### Flawed two-factor verification logic
Suppose a user attacker logs in with their account, then a cookie is assigned to this session which is used to track the verification code he uses. However if he manages to change this cookie to one for another account, then the verification code will be generated for another account. In this case if the code can be brute forced since for 4 digits you only have 10,000 requests to be made, 2FA can be easily bypassed.
