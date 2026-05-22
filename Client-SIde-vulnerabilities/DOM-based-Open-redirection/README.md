# 1. Executive Summary / Objective

**Objective:** The goal of this lab is to exploit a DOM-based open redirection vulnerability to successfully redirect a victim user to an external, attacker-controlled exploit server.

**Vulnerability Type:** DOM-based Open Redirection. 

# 2. Analysis & Discovery (The "Why")

**Discovery Process:** While navigating the application, I inspected the source code of the individual blog post pages to understand how client-side routing and navigation were being handled ,especially between blog post and home.

**The Clue:** I focused my attention on the "Back to Blog" link at the bottom of each post. I discovered that the application relies on client-side JavaScript to determine where this link should send the user. The script reads the current URL out of the browser (`window.location`), searches for a specific parameter (`url=`), and uses that value to set the new destination (`location.href`). Because the application takes user-controllable input (the source) and passes it directly into a dangerous execution function (the sink) without any validation, it became clear the application was vulnerable.

# 3. Exploitation (The "How")

By manipulating the URL string, I was able to force the vulnerable JavaScript to execute a redirect to a malicious domain.

## The Vulnerable Code:

This is the JavaScript snippet found handling the "Back to Blog" button logic:
```javascript
let returnUrl = /url=(https?:\/\/.+)/.exec(location);
if(returnUrl) {
    location.href = returnUrl[1];
} else {
    location.href = "/";
}
```

## Walkthrough:
- Accessed the lab application and opened the provided Exploit Server in a separate tab.
- Copied the Exploit Server URL (e.g., `https://exploit-0ad50001035ba28281d5430501a90047.exploit-server.net/exploit`).
- Navigated to an individual blog post on the target application.
- In the browser's address bar, appended this payload to the existing query string:
  ```plaintext
  &url=https://exploit-url.exploit-server.net/
  ```
  *(Example: `https://0a5700f60392a256818944d900a90024.web-security-academy.net/post?postId=5&url=https://exploit-0ad50001035ba28281d5430501a90047.exploit-server.net/exploit`)*
- Pressed Enter to reload so that `window.location` would update with injected parameter.
- Scrolled to bottom of blog post and clicked "Back to Blog" link.
- The vulnerable JavaScript executed regex, extracted malicious URL, passed it to `location.href`, redirecting browser and completing lab.

# 4. Remediation

To secure this code against DOM-based open redirection, developers should avoid dynamically setting redirection targets based on user-controllable data such as URL query strings.

If dynamic redirects are necessary, implement these defenses:
- **Use an Allowlist:** Instead of full URLs, accept identifiers like `?returnId=1` mapped securely on server or client side.
- **Validate Origin:** Parse URLs strictly in JavaScript; verify hostname matches expected domain before assigning `location.href`.
- **Avoid Sink Usage:** Use static relative paths for navigation (`href="/"`) rather than parsing current location dynamically.