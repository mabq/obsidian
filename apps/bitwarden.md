# Bitwarden

## 2FA

A 30-character random password makes brute-force guessing impossible, but it doesn't protect you from theft:

- Database Breaches (The "Out of Your Control" Factor)
  You can generate a pristine, unique password, but once you enter it into a website, its safety relies on their security. If the company misconfigures their server, gets hacked, or stores hashes insecurely, your password ends up on the dark web alongside millions of others. 2FA ensures that even if a database leaks your password tomorrow, no one can log in with it alone.

- AitM Phishing (Adversary-in-the-Middle)
  Modern phishing relies heavily on tricking you into typing your actual credentials into a fake domain that looks identical to the real site (e.g., app-login-auth.com instead of app.com). When you type your long password, the proxy server grabs it in real time. Standard passwords offer zero defense here; only 2FA (specifically hardware keys or passkeys) can stop this.

- Malware and Keyloggers
  If a device you use gets infected with infostealer malware, it doesn't matter how complex the password was—the malware simply reads the keystrokes or extracts stored session cookies straight from your browser memory.

- Session Hijacking & Credential Stuffing
  Automated bots test leaked password databases against thousands of services simultaneously. If you happen to reuse that long password even once (or a slightly modified version of it), automated tools will breach every account that shares it.

Two-Factor Authentication (2FA) acts as a physical circuit breaker. If an attacker gets your password, they still need something you have (like your phone, a security key, or an authenticator app) to get in.


## Regain access

If you loose access to the 2FA authentication device, regain access with the recovery code from:

1. Physical backup.
2. Emergency repo — requires decryptiong with `age`. Make sure you use a secure computer and delete everything afterwards.
