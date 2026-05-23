# Hacker101 – Micro-CMS v1

## 1. Goal and Objective
**Objective:** Capture all four flags in the Hacker101 Micro-CMS v1 challenge.
**Vulnerabilities Exploited:** Cross-Site Scripting (XSS), SQL Injection (SQLi), and Insecure Direct Object Reference (IDOR).

## 2. Analysis and Discovery
Upon launching the instance, I was directed to a simple Content Management System (CMS) homepage containing links to two existing pages: "Testing" and "Markdown."

Navigating through the application revealed a few critical clues:

- **Editable Content:** The application allows users to edit existing pages and create new ones. While testing the markdown functionality, I noticed some filtering was in place, which strongly hinted at potential XSS vulnerabilities if the filter could be bypassed.
- **Predictable URL Structure:** When editing or viewing pages, the URL included a distinct page parameter (e.g., `/page/edit/1`). This exposure of database IDs in the URL was a massive indicator to test for both SQL Injection (SQLi) and IDOR.

## 3. Capturing the Flags
### Flag 1: Stored XSS (Title Field)
I started by testing the input fields when creating a new page to see how the application handled raw HTML/JavaScript.

**Methodology:** I injected a basic JavaScript payload directly into the page's "Title" field.

**Payload:** `Hello<script>alert(1)</script>`

**Result:** The application failed to sanitize the title input. Upon saving and viewing the page, the script executed, revealing the first flag.

### Flag 2: Stored XSS (Description Field – Filter Bypass)
Next, I tested the description/body field of the page.

**Methodology:** I initially tried the standard `<script>alert(1)</script>` payload, but the application stripped the `<script>` tags, preventing execution. I needed to bypass this filter using HTML event handlers.

**Payload:** `<button onclick="alert(1)">Click Me</button>`

**Result:** The application allowed standard HTML tags like `<button>`. By embedding the JavaScript within the `onclick` attribute, clicking the rendered button executed the script and triggered the second flag.

### Flag 3: SQL Injection (URL Parameter)
Noticing the page IDs in the URL, I tested how the backend database handled unexpected input.

**Methodology:** I navigated to an existing page (e.g., `/page/edit/1`) and appended a single quote (`'`) to the end of the URL.

**Payload:** `/page/edit/1'`

**Result:** The single quote broke the backend SQL query syntax. Instead of failing gracefully, the application threw an error and dumped the third flag to screen, confirming SQL injection vulnerability.

### Flag 4: IDOR (Insecure Direct Object Reference)
The final vulnerability involved manipulating predictable URL structure to access unauthorized content.

**Methodology:** The default pages were numbered 1 and 2. When creating a new page, it was assigned ID 12. This indicated there were hidden or deleted pages (IDs 3 through 11) existing in database that were not linked on homepage. I manually enumerated URLs by changing ID number.

**Payload:** Manually navigating to `/page/edit/4`

**Result:** While most missing IDs returned a `404 Not Found` error, accessing page ID 4 successfully loaded a hidden, restricted page containing final flag. The application failed to verify if user was authorized to view that specific object.

## 4. Conclusion
the Micro-CMS v1 challenge is an excellent demonstration of why developers must treat all user input as untrusted.
The vulnerabilities found stem from three core failures:
- **Lack of Input Sanitization:** Allowing raw JavaScript to be stored and executed in browser (XSS).
- **Improper Database Queries:** Concatenating user input directly into database queries instead of using prepared statements (SQLi).
- **Missing Access Controls:** Trusting client requests for objects by ID without verifying authorization level on backend (IDOR).

Securing this app would require implementing strict output encoding for user-generated content, transitioning to parameterized SQL queries, and enforcing robust server-side access control checks for every request.