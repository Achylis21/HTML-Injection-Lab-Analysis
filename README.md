# HTML Injection & Weaponization Analysis

## Executive Summary
* **Severity:** Medium
* **Vulnerability Type:** Web Application Security / Input Validation Failure
* **Target Environment:** Local Sandbox Lab (HTML/JS)
* **Objective:** Demonstrate how insufficient input sanitation allows an attacker to manipulate webpage DOM structure, escalate the risk to a credential-harvesting phishing attack, and outline robust code-level remediation.

---

## Phase 1: Lab Architecture & Code Review
To understand the vulnerability, a localized guestbook dashboard was created. 

The application utilizes a JavaScript function to reflect user input back onto the screen. Below is the vulnerable snippet:
<img width="1919" height="952" alt="Main-Page-with-dev-console" src="https://github.com/user-attachments/assets/db720d48-3951-4ef4-a215-1302d1a9bf4c" />

```javascript
function submitComment() {
    let input = document.getElementById("userInput").value;
    // VULNERABILITY: innerHTML instructs the browser to parse strings as live HTML
    document.getElementById("commentDisplay").innerHTML = "Guest: " + input;
}
```

---

## Phase 2: Exploitation & Proof of Concept (PoC)

### Attack Scenario A: Basic Defacement
By inputting basic HTML ```<h1>``` headers, the application's visual structure was changed
<img width="1919" height="952" alt="Test-HTML-Injection-1" src="https://github.com/user-attachments/assets/1b5a15ca-3a50-4800-a187-8981ce94c3f6" />

### Attack Scenario B: Weaponization for Phishing
To demonstrate how an attacker can use this to provide a fake login page. 

```Code Injection used: 
<div style="style="padding: 10px; background-color: #f4f4f4; border-left: 5px solid #007BFF;"><b>Session Expired.</b>. Please click <button onclick="alert('User Hacked')">Here</button></div>
```

A malicious button was entered to demonstrate a possible redirection to a fake login page. Can also use malicious link
<img width="1918" height="953" alt="HTML-Injection-1-Phishing Button" src="https://github.com/user-attachments/assets/cefcfd1b-e23f-4d24-9710-4fdbbdab3c93" />

For this portfolio, it only shows an alert box
<img width="1919" height="951" alt="HTML-Injection-1-PhishingButton-Result" src="https://github.com/user-attachments/assets/f4980faa-76db-4da1-82a0-08a365745c4d" />

---

## Phase 3: Root Cause Analysis (The SOC Perspective)
The core failure stems from the misuse of the JavaScript `.innerHTML` property. The code treats the data as executable structure rather than plain, untrusted text. From a defensive standpoint, this can lead to Session Hijacking, Phishing, and if upgraded with script tags, full Cross-Site Scripting (XSS).

---

## Phase 4: Remediation & Verification
To mitigate this risk, the application must enforce strict data encoding. The `.innerHTML` property was stripped and replaced with `.textContent`, which forces the browser to treat all inputs strictly as an unexecutable text string.

### Secure Code Fix:
```javascript
function submitComment() {
    let input = document.getElementById("userInput").value;
    // SECURE: textContent neutralizes all injected HTML tags
    document.getElementById("commentDisplay").textContent = "Guest: " + input;
}
```

### Verification Testing:
When attempting the exact same payloads after applying the fix, the browser successfully neutralized the threat:
<img width="1919" height="953" alt="Resolution" src="https://github.com/user-attachments/assets/fa7eb93a-ed8f-491b-9485-b793d7ead249" />
