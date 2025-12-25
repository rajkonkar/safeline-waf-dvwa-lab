# SafeLine WAF + DVWA Lab

## Overview
This project documents a hands-on web security lab where **SafeLine Web Application Firewall (WAF)** is deployed as a **reverse proxy** in front of **Damn Vulnerable Web Application (DVWA)**. The goal was to prove WAF protections by generating real attack traffic from an attacker machine and validating **blocking + logging** in the SafeLine dashboard.

This lab was built end-to-end to demonstrate strong fundamentals in: reverse proxying, HTTPS/TLS termination, request inspection, rate limiting, bot mitigation, edge authentication, and IP-based access control.

---

## Architecture
    Kali Linux (Attacker)
            |
            v
    HTTPS (443)
    SafeLine WAF (Reverse Proxy + TLS Termination)
            |
            v
    HTTP (8081)
    DVWA (Apache on Ubuntu)

- WAF terminates HTTPS on port 443 and forwards to the backend DVWA service over HTTP on port 8081.
- The attacker machine only accesses DVWA through the WAF domain, so all traffic is inspected and enforced by SafeLine.

---

## Environment
- Kali Linux (attacker/testing machine)
- Ubuntu Server (hosting DVWA + SafeLine WAF)
- SafeLine WAF deployed via Docker (PRO trial used for advanced features)
- DVWA hosted on Apache listening on port **8081**
- Self-signed TLS certificate for the lab domain
- Local host resolution configured via `/etc/hosts` so `www.dvwa.local` points to the WAF IP

---

## What I Implemented & Tested

### 1) Reverse Proxy Setup (WAF in the traffic path)
- Created a SafeLine application for the domain `www.dvwa.local` on **443/HTTPS**
- Set upstream/backend to `http://<Ubuntu-IP>:8081`
- Ensured HTTP/80 was removed so only HTTPS/443 is exposed through the WAF
- Verified traffic was going through the WAF (not bypassing) by observing SafeLine session behavior and centralized logs

### 2) SQL Injection Detection & Blocking
- Accessed DVWA through the WAF at: `https://www.dvwa.local/DVWA/`
- Triggered SQL injection attempts in DVWA’s SQLi module (example payload: `' OR '1'='1`)
- Observed SafeLine returning an **Access Forbidden** block page
- Verified SafeLine logged the event under Attacks with:
  - Action: Blocked
  - Attack type: SQL Injection
  - Source IP: Kali IP
  - Full URL including the injected parameter

### 3) HTTP Flood Defense (Rate limiting + penalties)
- Enabled HTTP Flood/DoS protection in SafeLine
- Configured request-rate thresholds (requests within a time window) and penalty duration
- Generated high-rate traffic from Kali using load tools (e.g., `ab`, `siege`)
- Observed:
  - Rate limiting/threshold triggers
  - Temporary enforcement (penalty/ban applied)
  - Automatic unblocking after the configured duration
- Verified evidence in SafeLine logs showing the reason (e.g., “X requests within Y seconds”) and blocked request counts

### 4) Bot Protection (Challenge escalation)
- Flood testing also triggered SafeLine bot controls
- WAF escalated to **Anti-Bot Challenge** when traffic looked automated/non-browser
- Confirmed tools could not pass verification (no JS/browser fingerprint), resulting in blocked/challenged requests
- Verified bot-related telemetry (hits vs verified) in the SafeLine UI

### 5) WAF Authentication Gateway (Pre-app authentication)
- Enabled SafeLine **Auth Sign-In** for the DVWA application
- Configured simple username/password authentication at the WAF layer
- Verified that accessing DVWA from Kali prompted a SafeLine login page before any traffic was forwarded to DVWA
- Demonstrated layered security: WAF auth first, then DVWA’s own login

### 6) Custom Deny Rules (Block Kali by IP)
- Created a deny rule matching the Kali source IP
- Confirmed immediate blocking at the WAF (request stopped before reaching the backend)
- Verified deny-rule enforcement and visibility in logs

---

## Key Learnings (Security Fundamentals Demonstrated)
- A WAF provides **defense-in-depth**: signature detection (SQLi), rate limiting (flood), bot mitigation (challenge), and access controls (auth + IP deny) work together.
- HTTPS termination at the WAF enables centralized inspection and consistent enforcement.
- Rate limiting handles abusive traffic patterns; bot challenges raise the cost of automation.
- Edge authentication reduces exposure and prevents unauthenticated traffic reaching the app.
- Centralized logging is critical for security validation, troubleshooting, and reporting.

---

## Disclaimer
This lab was performed in an isolated environment for educational purposes only.
