# Same-Origin Policy & CORS Comments

## Comment #1: Adding diagrams and images to help explain the attack. 

To help enhance the learning experience of learners, we can add diagrams that help visualize each step of the process.  
For example, in [Senior Web Penetration Tester learning path -> Advanced XSS and CSRF Exploitation module -> Same-Origin Policy & CORS section](https://academy.hackthebox.com/module/235/section/2655), it starts with explaining the CSRF attacks.

```
Recap: Cross-Site Request Forgery (CSRF)
Cross-Site Request Forgery (CSRF) is a type of web attack where an attacker's payload forces a victim's browser to unintentionally perform actions in a vulnerable web application to which they are authenticated. CSRF attacks are typically performed by a payload on an attacker-controlled website, which sends cross-origin requests to the vulnerable web application. As such, the attack usually requires the victim to access the attacker-controlled website voluntarily or through other attack vectors, such as social engineering. In a successful CSRF attack, the cross-origin request is sent with the victim's session cookies and performs a change in the vulnerable web application.

As an example, consider the following scenario. The victim is an administrator of https://vulnerablesite.htb and is logged in to the site, i.e., the browser stores a valid session cookie. The site is not protected against CSRF attacks. The attacker controls a low-privilege account on the vulnerable web application and wants to execute a CSRF attack to obtain administrator privileges. When the victim accesses the attacker-controlled site https://exploitserver.htb, the site executes JavaScript code in the victim's browser that makes a cross-origin request to https://vulnerablesite.htb/promote?user=attacker. The browser sends the victim's session cookies with the cross-origin request such that the request is authenticated. Therefore, the web application promotes the attacker's user account to administrator, allowing the attacker to successfully execute a CSRF attack and obtain administrator privileges on the web application.

Although we will be creating the CSRF payloads manually in this module, there are tools that we can utilize for automatic payload generation, for instance the CSRF PoC generator here.
```

We can add a diagram like so to aid students:

![Alt text](./images/csrf.png?raw=true "CSRF diagram")


## Comment #2: Add more information on how to identify if CSRF protections are in place 

In the same section, there is a brief overview of CSRF tokens as a key defense mechanism. 

```
CSRF Tokens
CSRF Tokens are unique and random values that must be included in requests performing sensitive changes to the web application, for instance, when submitting HTML forms. The token must be unpredictable, so an attacker cannot know its value in advance. Furthermore, the web application needs to check the value of the CSRF token before performing the sensitive change. This prevents the attacker from constructing a cross-site request that the web application accepts. The token must be unpredictable, checked adequately by the backend, and not sent in a cookie, as otherwise, the CSRF token protection may be ineffective.

In our above example, the web application would only accept user promotion requests containing the username in the user GET parameter and the CSRF token in the csrf_token GET parameter, typically a hidden value in the HTML form. Since the CSRF token is a random value, the attacker cannot know the correct value, and thus, he is only able to construct a cross-origin request with an invalid CSRF token. If the web application checks the CSRF token correctly, the request will be rejected, so the attacker user account is not promoted to administrator privileges.
```

Showing learners exactly where to look for these tokens in HTTP requests and responses would help them spot whether a site has implemented any protection. Including screenshots like the following will reinforce how to locate and verify CSRF tokens:

![Alt text](./images/csrf_tokens.png?raw=true "CSRF token")

![Alt text](./images/csrf_tokens2.png?raw=true "CSRF token2")

## Comment #3: CSRF Token Implementations

In relation to number 2, having an idea of how CSRF tokens are implemented will help learners better understand if they have been implemented as well as if they are implemented properly. We can add information such as: 

When implementing CSRF tokens to protect sensitive actions, there are several ways to generate and verify tokens. As long as they are properly implemented and validated, they will effectively block CSRF attacks.  

### Synchronizer Token Pattern (Server-Side Token)

For applications that store session information, a common approach is the Synchronizer Token Pattern. In this pattern the server creates a random, unique token as soon as the user logs in or whenever they load a page that requires protection. This token is specific to the user’s session data and is stored on the server. 

Whenever the application renders an HTML form for state-changing actions (for example, updating account details), it places the CSRF token into a hidden input field. Once the user submits the form, the token is sent alongside the request, and the server compares it to the token stored in the session. If they match, the request is accepted. If not, it is rejected as a probable CSRF attempt.

![Alt text](./images/synchronizer_token.png?raw=true "Synchronizer Token diagram")

Some implementations generate a single token per session, which remains valid until the session ends. Other implementations rotate tokens more often, such as on every form load or request. Each approach has its own pros and cons. Per session tokens creates a longer window in which an attacker could exploit a stolen token. While a more frequent rotation of tokens can complicate the application's usability, such as if the user tries to revisit a page with an invalid token via the browser’s back button.
