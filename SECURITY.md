# Security Policy

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

Email: security@kotton.sh

Include:

- A description of the vulnerability and its potential impact
- Steps to reproduce or proof-of-concept code
- The version or commit you tested against, and the repository it is in

You'll receive an acknowledgment within 48 hours. We'll work with you on a
coordinated disclosure timeline — typically 90 days, shorter for critical issues
affecting user key material or encrypted data.

## What this repository is

This repository is the protocol: what crosses between Kotton's processes and what it
means. A finding here is one about the contract itself — a request whose meaning lets one
side reach what it should not, a verification a side may skip, an encoding two sides read
differently. A finding in an implementation (the system, the compositor, the vault, the
relay, a desktop) goes to the same address and names the repository.

A proposal to change the protocol is an RFC, in public. A vulnerability in the protocol is
not an RFC until it is fixed and shipped; it is a report to the address above.

## Scope

In priority order:

1. **Key material** — anything in the protocol that would let a key, a seed or a recovery
   phrase cross a boundary it should not
2. **Authority** — a request that acts on an authority its session does not hold, or a
   verification a side can skip
3. **Confinement** — a surface a guest can reach that it was not granted
4. **Encoding** — two sides reading one message differently

## Out of Scope

- Vulnerabilities in dependencies already disclosed upstream (report those upstream)
- Denial-of-service against a local daemon with no network exposure
- Issues that require physical access to an already-unlocked device

## What We Commit To

- We will not take legal action against researchers acting in good faith
- We will credit you in the release notes if you'd like
- We will not disclose your identity without your permission
