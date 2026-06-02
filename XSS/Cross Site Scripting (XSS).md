- Cross site scripting is a type of vulnerability which allows an attacker to compromise the interactions of a user with a vulnerable application.
- XSS works by allowing a attacker to command the execution of scripts on a users browsers by manufacturing malicious links for a vulnerable site.
- There exist three major types of XSS
	1. Reflected XSS
	2. Stored XSS
	3. DOM-based XSS
###### Reflected XSS
- A reflected XSS is a type of attack in which an application receives data from a request and and includes the data on the site in a unsafe manner allowing attackers to inject scripts into the application.
- This is most commonly done using URL parameters which allows a attack to manufacture of a malicious URL which can be used to attack users.
###### Stored XSS
- This type of XSS arises when a applications stores user submitted data in a unsafe manner, a application might store something from a unsafe source and later include it into a web request.
- The most common example might be of comments, if a comment allows a attacker to inject code into the application, and any user visits said page, the scripts can run on the users browser.
###### DOM based XSS
- This is the most "powerful" XSS attack as its attack surface stretches far and wide, a DOM(Document Object Model), is a website's hierarchical representation of elements on it.
- DOM allows Javascript to modify the page and allow for dynamic websites, DOM based XSS arises when javascript takes data from a attacker controlled value and sends it to a dangerous function called a sink.