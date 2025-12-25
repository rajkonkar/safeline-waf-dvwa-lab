# SafeLine WAF + DVWA Lab

This repo documents a home lab where SafeLine WAF is deployed as a reverse proxy in front of DVWA to demonstrate WAF protections.

## What I tested
- SQL Injection blocking + logging
- HTTP flood / rate limiting + bot challenge
- WAF authentication gateway (pre-app login)
- Custom deny rules (IP block)

## Architecture
Kali (attacker) → HTTPS (443) → SafeLine WAF → HTTP (8081) → DVWA (Apache)

## Disclaimer
Educational lab in an isolated environment only.
