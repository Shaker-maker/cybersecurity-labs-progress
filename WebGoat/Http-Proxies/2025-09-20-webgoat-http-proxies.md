# WebGoat Lab: HTTP Proxies


## Objective
Understand how to intercept, inspect and modify HTTP request using an intercepting proxy(OWASP: ZAP ProxY)
Learni to filter unnecessary internal request so only relevant traffic is captured.

## Steps Taken
1. **Configured Browser with ZAP Proxy**  
   - Set browser proxy to `localhost:8080`.
   - Verified traffic was visible in ZAP.

2. **Set Up Breakpoint Filter**  
   - Added a custom breakpoint in ZAP to intercept only POST requests.
   - Excluded WebGoat’s internal `.mvc` polling requests using regex:  
     ```
     ^POST\s(?!.*\.mvc).*
     ```

3. **Configured URL Filters**  
   - In ZAP filter tab:  
     - **Include Regex**: `.*WebGoat.*`  
     - **Exclude Regex**: `.*lesson.*\.mvc.*`

4. **Intercepted & Forwarded Requests**  
   - Triggered actions in WebGoat that sent POST requests.  
   - ZAP paused at the breakpoint.  
   - Examined request headers and body before forwarding to server.  
   - Confirmed `.mvc` background requests were not intercepted.

## Key Takeaways
- HTTP proxies like ZAP are essential tools in penetration testing.  
- Breakpoints allow fine-grained inspection and modification of requests.  
- Regex filters help avoid noise from internal/polling requests.  
- Every application may have unique internal endpoints; identify and exclude them to focus only on relevant requests.

## Commands / Regex Used
```regex
# Breakpoint rule (Request Header):
^POST\s(?!.*\.mvc).*

# URL Filters
Include: .*WebGoat.*
Exclude: .*lesson.*\.mvc.*
