# WebGoat — IDOR Lab (Insecure Direct Object Reference)

**Date completed:** 2025-10-19  
**Tools used:** Browser DevTools, Burp Suite (optional), WebGoat lab environment  
**Notes:** Lab-only work. This is a learning exercise; do not apply these techniques to real systems without authorization.

---

## Overview
This lab demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability. The goal is to authenticate as a normal user, observe how the app exposes object identifiers,
 and then manipulate those identifiers to view and modify other users' profiles.

I completed all parts of the lab. The first parts were straightforward; I used YouTube to help with the final two parts (intercepting and modifying requests).

---

## Lab walkthrough

### Part 1 — Authenticate first
- **Action:** Logged in using provided credentials:
  - `username: tom`
  - `password: cat`
- **Purpose:** Many access control flaws are exploitable only by an _authenticated but unauthorized_ user, so we authenticate first.

---

### Part 2 — Observe differences & hidden attributes
- **Action:** Viewed my profile and inspected the **raw server response** (Browser DevTools → Network → Response).
- **Observation:** The raw response contained attributes not visible in the rendered profile UI.
- **Answer (lab):** The two attributes present in the raw response but not shown in the profile were:
  1. `role` (or a similar permission/role field)
  2. `userid field
- **Lesson:** The UI sometimes hides data, but it can still be present in responses — always inspect raw responses.

---

### Part 3 — Guessing & predicting patterns (direct object reference)
- **Action:** Based on the app’s RESTful patterns, I checked likely URL patterns for direct object access.
- **Likely own-profile URL pattern (lab input):**

(For the lab you input a path starting with `WebGoat/` as instructed.)
- **Lesson:** RESTful apps often use predictable object identifiers in URLs. That’s the typical attack surface for IDOR.

---

### Part 4 — Playing with patterns (view & edit other profiles)
#### View another profile
- **Action:** Used the alternate path (the explicit profile URL) and changed the object identifier to another user (e.g., `{userId}` of "Buffalo Bill") via:
- Intercepting the request (Browser or Burp) and substituting the numeric/id part, **or**
- Manually constructing the URL in the browser.
- **Result:** I was able to view another user's profile information without proper authorization checks.

#### Edit another profile
- **Action:** Took a base request that edits profiles and modified:
- HTTP method (e.g., `GET` → `POST` or `PUT` depending on the app)
- URL path to target Buffalo Bill’s id
- Request body (payload) to change:
  - `role` → a lower number (lab hint: lower number = lower privilege in this app)
  - `color` → `red`
- **Result:** The app accepted the modified request and updated Buffalo Bill’s profile fields.
- **Note:** For these parts I watched a YouTube walkthrough to confirm request structure and payload formats — after that I repeated the steps in the lab.

---

## Example (sanitized) request/response
> **Note:** Remove or sanitize any sensitive info before publishing. Below is a lab-style example (not real-user data).

**Example GET (view profile):**

PUT /WebGoat/profile/3 HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Cookie: JSESSIONID=...; auth=...

{
"role": 1,
"color": "red",
"displayName": "Buffalo Bill"
}


---

## Findings & lessons learned
- IDORs are often trivially exploitable when server-side authorization is missing or insufficient.  
- Always enforce server-side ownership/authorization checks for object access and modifications.  
- Even if the UI hides fields or buttons, the backend may still perform actions without verifying the caller's permissions.  
- Inspect **raw responses** (not just the UI) — they frequently reveal hidden data or ID patterns.  
- For testing, intercepting/modifying HTTP requests (browser devtools or Burp Suite) is essential.

---

## Remediation suggestions (for developers)
1. **Server-side authorization:** For every endpoint that accesses or modifies objects, verify the caller is permitted to perform the action on the specific object (ownership or access control lists).  
2. **Avoid predictable object identifiers:** Use unguessable IDs (UUIDs or sufficiently random identifiers) or map public IDs to internal IDs after authorization. Note: ID obfuscation alone is not enough — always enforce server-side checks.  
3. **Principle of least privilege:** Make sure roles/permissions are well-defined and checked on the server.  
4. **Logging & monitoring:** Log suspicious access patterns (many different IDs accessed by a single user) and alert on anomalies.  
5. **Automated tests:** Add unit/integration tests that attempt to access/modify objects belonging to other users and assert denial.

---

