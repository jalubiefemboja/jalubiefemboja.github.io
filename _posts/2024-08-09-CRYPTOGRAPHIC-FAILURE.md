---
title: Use of Weak Hash
category: General
---

## Intro ##
The second most common web application vulnerability according to the Open Web Application Security Project (OWASP) ​[1](#1)​ is Cryptographic Failures, which is a “broad symptom rather than a root cause”, that refers to either a failure of cryptography, or lack thereof ​[2](#2). To conduct a productive investigation into this area, we must limit our scope to a direct descendant of this family of vulnerabilities, a palpable problem that we can identify in the field and attack to demonstrate its vulnerability.



The specific vulnerability that maps onto this category, and the one I would like to illustrate in this article is “Use of Weak Hash” ​[3](#3)​. In a word, the vulnerability of a weak hash originates in its inability to prevent an attacker from determining the original input once the function has been applied to it.



As the Common Weakness Enumeration (CWE) states, a hashing algorithm takes a value of arbitrary size and maps it onto a fixed-size output such that the output is irreversible, and the same input produces the same output every time, making it deterministic. The cryptographic strength of the hash function bases itself on its deterrence against determining the original value from the output (preimage attack), determining an input that can produce the same output as another input (second preimage attack), or multiple inputs producing the same output (birthday attack).  



Unfortunately, the infinite number of inputs paired with the finite permutations that can fit within our fixed-size output makes it a guarantee that with time an input will no longer map to a unique output, and a “collision” will occur. The mathematical impossibility of a collision-free hash function means that it is inevitable a collision attack will occur eventually, and one of the most widely-known instances of this is with the MD5 algorithm ​ [4](#4) .



The MD5 message-digest algorithm, devised in 1991, effectively became obsolescent in 2004 when it was found a collision attack could be carried out within hours on a single PC ​[5](#5), meaning that the computational complexity of the algorithm could no longer stand up to modern computational efficiency, or in mathematical terms a collision with a 128-bit hash value could be achieved in less than 2128 tries, rendering the algorithm cryptographically broken.



Because one of the most common applications of hashing algorithms is password hashing, unauthorised access could lead to data breach, financial losses, exposure of sensitive data, and so on, impacting everyone from individual users to international corporations and governmental organisations. For less security-intensive uses, such an algorithm could suffice, which is why MD5 is still about in the case of authenticating digital signatures or downloads via a checksum for example. The solution henceforth has been the implementation of increasingly stronger algorithms such as SHA-256, which produces a hash value twice as big and therefore yields a higher collision resistance.


## Practical ##
In the following PortSwigger exercise, there is an example of an MD5-based hashing algorithm which is being used to store passwords in a way that is inappropriate for the context. How this is inappropriate will quickly become apparent as we walkthrough it. ​[4](#4)

![wiener login]({{site.baseurl}}/assets/images/Picture20.png)


When we select “Stay logged in”, we see our selection reflected in the "Cookie" header in the subsequent HTTP request. Briefly, an HTTP cookie is a “small piece of data that a server sends to a user’s web browser”​ [5]​ to identify them, record choices and apply their preferences to a web session. Once a cookie is created, a specific user’s preferences or session information e.g. view history, shopping basket, so on, will be remembered for as long as the cookie lasts – in this case, their “stay-logged-in” status.

![wiener login]({{site.baseurl}}/assets/images/Picture19.png)

In the Inspector tab, we see that the cookie data can be decoded from Base64 to a value constructed from the login username followed by an incomprehensible alphanumeric value. Using CrackStation ​[6](#6)​, we can determine that the plaintext result of this value yields the password we used to sign into peter’s account, which exposes a massive vulnerability in the website’s security given that we can now determine username-password pairs simply from acquiring a victim’s cookie.

![wiener login]({{site.baseurl}}/assets/images/Picture18.png)

Upon probing the website further, we learn that the same website is also vulnerable to stored cross-site scripting (XSS) via the comment functionality, which we can then leverage to send cookies to our exploit server, given that visiting the blog post will now include scripts in future HTTP responses and execute in the victim’s browser ​[7](#7)​.

![wiener login]({{site.baseurl}}/assets/images/Picture17.png)
![wiener login]({{site.baseurl}}/assets/images/Picture16.png)
![wiener login]({{site.baseurl}}/assets/images/Picture15.png)
![wiener login]({{site.baseurl}}/assets/images/Picture14.png)
![wiener login]({{site.baseurl}}/assets/images/Picture13.png)
![wiener login]({{site.baseurl}}/assets/images/Picture12.png)

Now that the cookie-stealing script has been stored on the web server, visitors to the page will have their cookie sent to the exploit server. As we ascertained earlier, the decoded cookie format reveals the username and an MD5 hash of the password, which we can paste into CrackStation to yield the value “onceuponatime”. Predictably, we can gain access to carlos’s account!

![wiener login]({{site.baseurl}}/assets/images/Picture11.png)

## References ##
## [1] ##
Open Web Application Security Project, "OWASP Top Ten | OWASP Foundation," [Online]. Available: https://owasp.org/www-project-top-ten/. [Accessed 3 June 2024].
## [2] ##
Open Web Application Security Project, "A02 Cryptographic Failures - OWASP Top 10:2021," 2021. [Online]. Available: https://owasp.org/Top10/A02_2021-Cryptographic_Failures/. [Accessed 2 June 2024].
## [3] ##
Common Weakness Enumeration, "CWE-328: Use of Weak Hash (4.14)," 29 February 2024. [Online]. Available: https://cwe.mitre.org/data/definitions/328.html. [Accessed 15 March 2024].
## [4] ##
PortSwigger, "Lab: Offline password cracking | Web Security Academy," [Online]. Available: https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking. [Accessed 15 March 2024].
## [5] ##
Mozilla, "Using HTTP cookies - HTTP | MDN," 23 February 2024. [Online]. Available: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies. [Accessed 22 March 2024].
## [6] ##
CrackStation, "CrackStation - Online Password Hash Cracking - MD5, SHA1, Linux, Rainbow Tables, etc.," 27 May 2019. [Online]. Available: https://crackstation.net/. [Accessed 2 June 2024].
## [7] ##
PortSwigger, "What is stored XSS (cross-site scripting)? Tutorial & Examples | Web Security Academy," [Online]. Available: https://portswigger.net/web-security/cross-site-scripting/stored. [Accessed 2 June 2024].
## [8] ##
Common Weakness Enumeration, "CWE-327: Use of a Broken or Risky Cryptographic Algorithm (4.14)," 29 February 2024. [Online]. Available: https://cwe.mitre.org/data/definitions/327.html. [Accessed 15 March 2024].
