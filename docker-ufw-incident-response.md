<!-- ============================================================ -->
<!-- FILE: docker-ufw-incident-response/README.md               -->
<!-- ============================================================ -->
 
# Docker/UFW Firewall Bypass — Incident Response
 
**TL;DR:** Identified and resolved a production security incident where Docker bypassed UFW firewall rules, inadvertently exposing a CouchDB port publicly on a university server. Remediated, added audit scripts, and built a git pre-commit hook to prevent recurrence.
 
## What Happened
 
Docker's default behaviour modifies iptables rules directly, bypassing UFW. A CouchDB container was running with a published port that became publicly accessible, unintended by the firewall configuration.
 
## Response Steps
 
1. **Identification** — confirmed public exposure via port scan
2. **Containment** — rebound all services to local IP (`127.0.0.1`)
3. **Remediation** — updated Docker daemon config to prevent iptables manipulation
4. **Prevention** — wrote audit script to check for exposed ports; added git pre-commit hook to flag docker-compose files publishing ports without explicit bind address
## Key Lesson
 
Docker and UFW don't play well together by default. Docker's iptables integration bypasses UFW entirely — `ufw status` can show a port as blocked while it's actually open to the internet.
 
## Tools & Technologies
 
`Docker` `UFW` `iptables` `Bash` `CouchDB`
 
