# Security Policy

We take the security and privacy of our users seriously. If you believe you have
found a security vulnerability in any Prinsur product or service, we want to
hear about it.

## Reporting a vulnerability

Email **support@prinsur.com** with the subject line prefixed `[SECURITY]`.

Please do not open a public issue, pull request or discussion for security
reports.

Include as much of the following as you can:

- The product or endpoint affected
- A description of the issue and why you believe it is a security problem
- Steps to reproduce, or a proof of concept
- The impact you believe an attacker could achieve
- Any logs, screenshots or request captures that help us confirm the issue

Reports in English or Traditional Chinese are both fine.

## What to expect

- We aim to acknowledge your report within 3 business days.
- We aim to give you an initial assessment within 10 business days.
- We will keep you updated while we work on a fix, and let you know when it
  ships.
- We are happy to credit you publicly once the issue is resolved. Tell us how
  you would like to be named, or let us know if you prefer to stay anonymous.

We do not currently run a paid bug bounty program.

## Scope

Our products are delivered as hosted services, so only the currently deployed
version is supported. There are no older releases to patch.

Out of scope:

- Findings from automated scanners without a demonstrated impact
- Missing security headers or cookie flags with no exploitable consequence
- Denial of service, volumetric testing or resource exhaustion
- Social engineering, phishing or physical attacks against our staff or users
- Vulnerabilities in third-party services we do not operate. Please report
  those to the relevant vendor.

## Testing guidelines

When investigating an issue, please:

- Only test against accounts and data that belong to you
- Stop as soon as you have confirmed a vulnerability, and do not pivot further
  into our systems
- Never access, modify, exfiltrate or retain another person's data, including
  conversation transcripts
- Give us a reasonable window to fix the issue before disclosing it publicly

We will not pursue legal action against researchers who follow this policy in
good faith.
