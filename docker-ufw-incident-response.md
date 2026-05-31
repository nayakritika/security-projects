# Docker/UFW Firewall Bypass — Incident Response

**TL;DR:** Identified and resolved a production security incident where Docker bypassed UFW firewall rules, exposing a CouchDB port publicly on a university server. Remediated, added audit scripts, and built a git pre-commit hook to prevent recurrence.

## What Happened

Docker's default behaviour modifies iptables rules directly, bypassing UFW entirely. A CouchDB container was running with a published port that became publicly accessible — unintended by the firewall configuration. `ufw status` showed the port as blocked while it was actually open to the internet.

## Response Steps

1. **Identification** — confirmed public exposure via port scan
2. **Containment** — rebound all services to local IP (`127.0.0.1`)
3. **Remediation** — updated Docker daemon config to prevent iptables manipulation
4. **Prevention** — wrote audit script to detect exposed ports; added git pre-commit hook to flag docker-compose files publishing ports without an explicit bind address

## Key Lesson

Docker and UFW don't play well together by default. Docker's iptables integration bypasses UFW entirely — you can't rely on `ufw status` alone to verify your exposure surface when Docker is running.

## Tools & Technologies

`Docker` `UFW` `iptables` `Bash` `CouchDB`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Personal project / incident response*
