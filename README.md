# Experiment 10: Burp Suite as a Proxy for DVWA Login Page

## Aim
To configure Burp Suite as an intercepting proxy and use it to intercept, modify, and replay HTTP requests of the DVWA login page in the lab setup (DVWA running in Docker on Kali Linux).

## Apparatus Required
**Software**
- Kali Linux (physical or VM)
- Docker
- DVWA Docker image (`vulnerables/web-dvwa`)
- Burp Suite Community Edition (pre-installed on Kali)
- Burp Suite's built-in browser (Chromium)

## Procedure

### A. Set up DVWA
1. Install and start Docker:
   ```
   sudo apt update
   sudo apt install -y docker.io
   sudo systemctl start docker
   ```
2. Pull and run DVWA (`--rm` removes the container when it stops):
   ```
   sudo docker pull vulnerables/web-dvwa
   sudo docker run --rm -d --name dvwa -p 80:80 vulnerables/web-dvwa
   ```
   If port 80 is busy, use `-p 8081:80`.

### B. Configure Burp Suite
3. Open Burp Suite (temporary project, default settings). Confirm the proxy listener is `127.0.0.1:8080`.
4. Go to **Proxy → Intercept → Open browser** (built-in browser, already proxied).
5. In that browser open `http://127.0.0.1`, log in with `admin` / `password`, click **Setup / Reset DB → Create / Reset Database**, log in again, and set **DVWA Security** to **Low**.

### C. Intercept and modify
6. Log out, then turn **Intercept is on**.
7. Submit `admin` / `password` on the login page. Observe the `POST /login.php` request (`username`, `password`, `Login`, `user_token`).
8. Change the password to `wrong123` and click **Forward**. Note the response.

### D. Replay
9. In **Proxy → HTTP history**, right-click the login request → **Send to Repeater**.
10. In **Repeater**, click **Send** with correct credentials, then with `admin` / `wrong123`. Compare the responses.
11. Turn intercept off, close the browser, and stop DVWA: `sudo docker stop dvwa`.

> **Notes**
> - `user_token` is single-use. If a replay misbehaves, capture a fresh request.
> - With `--rm` the database resets every run, so repeat the reset in step 5.
> - On port 8081, use `http://127.0.0.1:8081`.

## Output
| Request Sent | Credentials | HTTP Status | Response / Behaviour |
|---|---|---|---|
| Original (captured) | admin / password | 302 Found | Redirects to `index.php`, login successful |
| Modified (replayed) | admin / wrong123 | 302 Found | Redirects back to `login.php`, "Login failed" |

*(Attach screenshots of: Docker `docker ps` output, Burp intercept tab, HTTP history, Repeater request/response.)*

## Result
DVWA was deployed using Docker on Kali Linux and Burp Suite was configured as a proxy on `127.0.0.1:8080`. HTTP requests of the DVWA login page were intercepted, modified, and replayed using Proxy and Repeater, and the server's responses differed according to the values submitted.

## Viva Questions and Answers
**1. What is Burp Suite?**
A web application security testing platform. Its proxy sits between the browser and the server so requests and responses can be inspected and modified.

**2. What is DVWA?**
Damn Vulnerable Web Application, a deliberately insecure PHP/MySQL app used to practice web security testing legally.

**3. What is Docker?**
A platform that runs applications in lightweight isolated containers, so DVWA can be started without installing Apache, PHP, and MySQL manually.

**4. What does `--rm` do in `docker run`?**
It automatically removes the container when it stops.

**5. What do `-d` and `-p 80:80` mean?**
`-d` runs the container in the background; `-p 80:80` maps host port 80 to container port 80.

**6. What is an intercepting proxy?**
A proxy that holds HTTP(S) traffic so the tester can view, edit, forward, or drop it before it reaches the server.

**7. What is the default Burp proxy listener?**
`127.0.0.1:8080`.

**8. Difference between Proxy, Repeater, and Intruder?**
- **Proxy:** captures and modifies live traffic.
- **Repeater:** manually resends a single request with edits to compare responses.
- **Intruder:** automates many requests with payloads (e.g., brute force, fuzzing).

**9. Why is the Burp CA certificate needed, and why not for the built-in browser?**
It lets Burp decrypt and inspect HTTPS traffic without certificate errors. The built-in browser is preconfigured to use the proxy and trust Burp's certificate, so no manual setup is required.

**10. What is `user_token` in DVWA?**
An anti-CSRF token that changes on every page load and is validated by the server on submission.

**11. Why does a replayed request sometimes fail?**
Because the single-use `user_token` has already been consumed.

**12. What do Forward and Drop do?**
Forward sends the request to the server; Drop discards it.

**13. Which HTTP method does the login form use?**
POST, with credentials in the request body.

**14. Why should DVWA be run only in an isolated lab?**
It is intentionally vulnerable and could be exploited if exposed to a network or the internet.

**15. Is it legal to test any website with Burp Suite?**
No. Testing is legal only with explicit authorization, such as on your own lab setup.
