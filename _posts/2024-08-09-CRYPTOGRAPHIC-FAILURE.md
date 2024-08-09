---
title: Use of Weak Hash
category: General
---

The second most common web application vulnerability according to the Open Web Application Security Project (OWASP) ​[1]​ is Cryptographic Failures, which is a “broad symptom rather than a root cause”, that refers to either a failure of cryptography, or lack thereof ​[2]​. To conduct a productive investigation into this area, we must limit our scope to a direct descendant of this abstract family, a palpable problem that we can identify in the field and attack to demonstrate its vulnerability.



The specific vulnerability that maps onto this category, and the one I would like to illustrate today is “Use of Weak Hash” ​[3]​. In a word, the vulnerability of a weak hash originates in its inability to prevent an attacker from determining the original input once the function has been applied to it.



As the CWE (Common Weakness Enumeration) states, a hashing algorithm takes a value of arbitrary size and maps it onto a fixed-size output such that the output is irreversible, and the same input produces the same output every time, making it deterministic. The cryptographic strength of the hash function bases itself on its deterrence against determining the original value from the output (preimage attack), determining an input that can produce the same output as another input (second preimage attack), or multiple inputs producing the same output (birthday attack).  



Unfortunately, the infinite number of inputs paired with the finite permutations that can fit within our fixed-size output makes it a guarantee that an input will no longer map to a unique output, and a “collision” will occur. The mathematical impossibility of a collision-free hash function means that it is inevitable a collision attack will occur in time, and one of the most well-known instances of this is with the MD5 algorithm ​[4]​.



The MD5 message-digest algorithm, devised in 1991, effectively became obsolescent in 2004 when it was found a collision attack could be carried out within hours on a single PC ​[5]​, meaning that the computational complexity of the algorithm could no longer stand up to modern computational efficiency, or in mathematical terms a collision with a 128-bit hash value could be achieved in less than 2128 tries, rendering the algorithm cryptographically broken.



Because one of the most common applications of hashing algorithms is password hashing, unauthorised access can lead to data breach, financial losses, and so on.  





To clarify, a hashing function is cryptographic, but not “encryption” as such.  

The issue with MD5-based hashing algorithms is it makes it determinable, “cryptographically broken”



In the following PortSwigger exercise, there is an example of an MD5-based hashing algorithm which is  



To do this, let’s do a PortSwigger exercise. ​[4]​
![test]({{site.baseurl}}/assets/images/Picture1.png)
A screenshot of a login form

Description automatically generated

When we select “Stay logged in”, we see our selection reflected in the "Cookie" header in the subsequent HTTP request. Briefly, an HTTP cookie is a “small piece of data that a server sends to a user’s web browser”​ [5]​ to identify them, record choices and apply their preferences to a web session. Once a cookie is created, a specific user’s preferences or session information e.g. view history, shopping basket, so on, will be remembered for as long as the cookie lasts – in this case, their “stay-logged-in” status.











In the Inspector tab, we see that the cookie data can be decoded from Base64 to a value constructed from the login username followed by an incomprehensible alphanumeric value. Using CrackStation ​[6]​, we can determine that the plaintext result of this value yields the password we used to sign into peter’s account, which exposes a massive vulnerability in the website’s security given that we can now determine username-password pairs simply from acquiring a victim’s cookie.



Upon probing the website further, we learn that the same website is also vulnerable to stored cross-site scripting (XSS) via the comment functionality, which we can then leverage to send cookies to our exploit server, given that visiting the blog post will now include scripts in future HTTP responses and execute in the victim’s browser ​[7]​.  

A close up of a text

Description automatically generated

A screenshot of a computer

Description automatically generated





A screenshot of a computer

Description automatically generatedNow that the cookie-stealing script has been stored on the web server, visitors to the page will have their cookie sent to the exploit server. As we ascertained earlier, the decoded cookie format reveals the username and an MD5 hash of the password, which we can paste into CrackStation to yield the value “onceuponatime”. Predictably, we can gain access to carlos’s account  
