# Cybersecurity Architecture Lab — Security Architecture Diagram

This folder contains two security architecture exercises:

1. [E-commerce application lab](#scenario) — the fundamentals of security
architecture diagramming: security zones, trust boundaries, and control
placement for a simple e-commerce web application.
2. [Jackson Corporation project](#jackson-corporation--network-security-assessment--architecture-design) —
a network security assessment with written recommendations, turned into a
full multi-layered architecture diagram.

## Scenario

A public-facing e-commerce application with:

* A website where customers browse and shop
* An administrative portal for staff to manage products and orders
* A database storing customer information and orders

## Diagram

!\[Intermediate security architecture](intermediate\_security\_architecture.svg)

[View live diagram (read-only)](https://viewer.diagrams.net/?url=https://raw.githubusercontent.com/CyberRay007/my-portfolio/main/cybersecurity-architecture-lab/intermediate_security_architecture.drawio)

The diagram segments the application into five trust zones, each separated by
a dedicated control:

|Zone|Trust level|Contains|
|-|-|-|
|Internet zone|Untrusted|External users|
|—|—|Perimeter firewall + WAF (blocks common web attacks)|
|DMZ|Semi-trusted|Load balancer, web servers (TLS termination)|
|—|—|Internal firewall (app-tier allowlist only)|
|Application zone|Trusted|App server, auth / API gateway|
|—|—|Data-tier firewall (encrypted connections only)|
|Data zone|Highly trusted|Primary database, backup / replica (encrypted at rest)|
|Management zone|Out-of-band|Bastion host (MFA), SIEM / logging|

**Design principles applied:**

* Every hop between zones crosses a security control — no direct access
across a trust boundary
* Sensitive data (the database) sits furthest from the untrusted zone
* Administrative access is routed through a bastion host rather than directly
into the application tier
* Centralized logging (SIEM) aggregates events from all zones for monitoring

## Source file

The editable `.drawio` source lives in this folder alongside the rendered
image. The "View live diagram" link above opens it through draw.io's
read-only viewer, so anyone can explore the full diagram without being able
to edit or save changes — only someone with push access to this repo can
change the source.

\---

# Jackson Corporation — Network Security Assessment \& Architecture Design

I assessed a fictional company's network and web application, wrote findings
and recommendations, then translated those recommendations into a target-state
security architecture diagram.

## Scenario

Jackson Corporation had a flat network and a web application with several
common weaknesses:

* A single firewall handling both internet and internal traffic
* A web server sitting inside the internal network
* No centralized security monitoring or breach detection
* Remote access over public networks using web-based tools
* A waterfall development process with little security focus
* OWASP-style flaws: weak passwords, plain-text sensitive data, no input
validation, and detailed error messages shown to users

## Diagram

!\[Jackson Corporation security architecture](Jackson\_Corporation.png)[View live diagram (read-only)](https://viewer.diagrams.net/?url=https://raw.githubusercontent.com/CyberRay007/my-portfolio/main/cybersecurity-architecture-lab/Jackson_Corporation.drawio)

## Part 1 — Infrastructure assessment

|Component|Finding|Recommendation|
|-|-|-|
|Firewalls|One firewall filtered both external and internal traffic|Two-barrier design with a DMZ: external firewall facing the internet, internal firewall guarding company data|
|Web servers|Web server sat on the internal network|Move it into an isolated DMZ between the two firewalls|
|Network monitoring|Wired and wireless devices all interconnected, no central logging|Zero Trust: isolate guest/wireless networks and segment wired networks by department (e.g. HR vs Marketing)|
|Breach detection|Monitoring focused on performance, not security|SIEM + XDR, continuous threat hunting, instant alerts to the IT team|
|Remote work|Web-based access over public or unsecured networks|Mandatory VPN, approved-software-only devices, IAM with least privilege, 15-minute inactivity timeout with re-verification|
|Software development|Waterfall process, infrequent code reviews|Automated code scanning, secure coding training, continuous "security first" workflow|

## Part 2 — Web application (OWASP) assessment

|Issue|Finding|Recommendation|
|-|-|-|
|Weak password policy|Simple, short passwords allowed|12–14+ character complexity rules, common-password blocklist, MFA|
|Unencrypted sensitive data|Card numbers and personal identifiers stored in plain text|Encryption at rest, HTTPS/TLS in transit, restricted database access|
|No input validation|Fields accepted unrestricted input|Strict input validation and sanitization on all entry points|
|Detailed error messages|Technical and database details shown to users|Generic public messages; detailed errors go to a secure internal log|

## Part 3 — Architecture

The diagram separates the environment into zones, each divided by a dedicated
control:

|Zone|Trust level|Contains|
|-|-|-|
|Internet zone|Untrusted|Customers, remote employees, threat actors|
|—|—|External firewall (outer barrier)|
|DMZ|Semi-trusted|WAF, IDS/IPS, web server, VPN gateway|
|—|—|Internal firewall (inner barrier, approved traffic only)|
|Internal zone|Trusted|Application servers, IAM + MFA, DevSecOps pipeline, VLAN-segmented office network (HR, Marketing, IT/Dev), company laptops, SIEM + XDR, SOC|
|Protected data zone|Highly restricted|Database firewall / ACL, encrypted database servers|
|Guest / wireless zone|Isolated|Guest devices — internet access only|

**Key traffic flows:**

* **Customer request:** Internet → external firewall → WAF → IDS/IPS → web server → internal firewall → application servers → database firewall → database
* **Remote employee:** Internet → external firewall → VPN gateway (MFA) → internal firewall → IAM → approved resources
* **Monitoring:** logs from every zone flow into the SIEM/XDR, which sends instant alerts to the security team
* **Guest Wi-Fi:** internet access only, blocked from all internal resources

**How each recommendation appears in the diagram:**

|Recommendation|Where it shows up|
|-|-|
|Two-barrier firewall + DMZ|External and internal firewalls around the DMZ|
|Relocated web server|Web server placed in the DMZ|
|Zero Trust segmentation|HR / Marketing / IT VLANs and the isolated guest zone|
|SIEM, XDR, threat hunting, alerts|SIEM + XDR collecting logs from all zones, feeding the SOC|
|Secure remote access|VPN gateway, IAM + MFA, approved-software laptops, 15-minute timeout|
|Secure development|DevSecOps pipeline deploying to the application servers|
|Weak passwords|IAM + MFA with strong password rules|
|Unencrypted data|AES-256 at rest, TLS on every link, database firewall|
|No input validation|WAF at the edge plus validation and sanitization on application servers|
|Detailed errors|Generic public errors; detailed errors sent to internal logs|

**Design principles applied:**

* Defence in depth: the database sits behind three firewall layers
* External users can never reach the internal network directly
* Every zone boundary has a firewall or security control
* Every connection is labeled with its protocol and encryption status
* Centralized logging (SIEM) covers all zones

## Reflection

* Drawing the architecture made the recommendations easier to understand than a
written list, because you can see where each control sits and how traffic
passes through it.
* The biggest impact comes from zone separation: the DMZ and second firewall
mean one compromised web server no longer exposes the internal network.
* For non-technical executives, I would explain it as a building: a front gate
(external firewall), a reception area (DMZ), a locked inner door (internal
firewall), a vault (database), and cameras everywhere (SIEM).
* Trade-offs to plan for: higher cost, added management complexity, staff
training, and tuning the SIEM and WAF to reduce false alerts.

## Download and improve this diagram

This diagram is open for anyone to download, edit, and build on.

**Option 1 — Open and edit it right now (no install)**

[Open in the draw.io editor](https://app.diagrams.net/?url=https://raw.githubusercontent.com/CyberRay007/my-portfolio/main/cybersecurity-architecture-lab/Jackson_Corporation.drawio)

This opens your own working copy in the browser. The original in this repo is
not changed. When you finish, use **File > Save as** or **File > Export as**
(PNG, SVG, PDF) to keep your version.

**Option 2 — Download the file**

1. Open [`Jackson\_Corporation.drawio`](Jackson_Corporation.drawio) in this folder.
2. Click the **Download raw file** button (top right of the file view).
3. Open it in [draw.io](https://app.diagrams.net) (**File > Open from > Device**) or the draw.io desktop app. It also works in VS Code with the "Draw.io Integration" extension.

**Option 3 — Contribute your improvements**

1. Fork this repository.
2. Edit `Jackson\_Corporation.drawio` and re-export the `.svg` image.
3. Open a pull request with a short note on what you changed and why.

**Ideas to build on**

* Add a cloud or hybrid zone (AWS, Azure) for backups and workloads
* Add a Zero Trust access broker or ZTNA in place of the traditional VPN
* Add email security, DNS filtering, and a proxy for outbound traffic
* Add a backup, disaster recovery, and incident response flow
* Add a patch management and vulnerability scanning server
* Map each control to a framework such as NIST CSF or CIS Controls
* Add a high availability pair for the firewalls and web server

**Usage:** free to use for learning, teaching, and portfolio projects. Please
credit the original author (Raymond Favour Joshua, CyberRay007) and link back
to this repository.

*This is a training project based on a fictional company scenario.*

