# Burp Suite — WebGoat `hijack_cookie` lab

**Purpose:** Document the exact steps you used in Burp Suite to intercept a request, test a predicted cookie in Repeater, and then configure an Intruder attack using a **Numbers** payload to brute-force the timestamp range. This playbook records a **Burp-only** workflow.

---

## 1) Summary

* **Target:** WebGoat lesson — predict/guess `hijack_cookie` value to access another user's authenticated session.
* **Tools used:** Burp Suite (Proxy, Repeater, Intruder).
* **Payload type used:** **Numbers** payload (numeric range on the last digits of the timestamp).

---

## 2) Environment & prerequisites

* Burp Suite installed and running.
* Browser configured to use Burp as HTTP proxy.
* WebGoat running and reachable by browser.

---

## 3) Capture and modify request (Repeater)

1. Start Burp and ensure **Proxy → Intercept is on**.
2. In your browser, log into WebGoat and capture a request containing `hijack_cookie` values for two valid sessions. Example observed:

   ```
   hijack_cookie=3640788706117592389-1758557466121; Path=/WebGoat;
   Set-Cookie: hijack_cookie=3640788706117592387-1758557450409; Path=/WebGoat;
   ```
3. Notice: one cookie ends with `...87` and another with `...89`. This means the missing victim’s cookie should be `...88`.
4. Manually craft a new candidate cookie by changing the session ID to 88 and keeping the timestamp part from 87 as a starting point:

   ```
   hijack_cookie=3640788706117592388-1758557450409;
   ```
5. Add this cookie to the request in **Repeater** and send to test manually.

---

## 4) Configure Intruder attack (Numbers payload)

### a) Send request to Intruder

* Right-click the modified request in Repeater → **Send to Intruder**.

### b) Mark positions

* In **Positions**, click **Clear §**.
* Highlight only the **last five digits of the RHS timestamp** (e.g., `50409`) and click **Add §**.
* This ensures only the varying portion is brute-forced.

### c) Configure payloads

* In **Payloads**:

  * **Payload type:** Numbers
  * **From:** `50409`
  * **To:** `66121`
  * **Step:** `1`
* Burp will generate every number between 50409 and 66121 and substitute into the request.

### d) Run the attack

* Make sure **Intercept is OFF**.
* Click **Start attack**.
* Monitor responses for anomalies (different length or matching a success string).

---

## 5) Confirm hijack

* Copy the successful cookie candidate back into **Repeater** or directly into your browser cookies.
* Verify that you now have access to the victim’s authenticated session.

---

## 6) Logging & results template

* **Date:** 2025-09-22
* **Target:** WebGoat — `A1: Broken Access Control` (session hijack exercise)
* **Successful cookie:** record exact value found
* **Method:** Burp Proxy (capture) → Repeater (manual crafting) → Intruder (Numbers payload on last 5 digits)
* **Payload range:** 50409 → 66121

---

## 7) Lessons learned

* Predictable session IDs (`...87`, `...89`) revealed the missing victim session ID (`...88`).
* RHS timestamp portion only varied in the last five digits, making a numeric brute-force practical.
* Always use cryptographically secure random values for cookies to prevent such predictable attacks.

---

*End of Burp-only workflow document (Numbers payload method, targeting last 5 digits).*
