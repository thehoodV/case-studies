# 🛡️ Case Study — Origin Server Exposure Behind CloudFront

## 📌 Overview

During a real-world reconnaissance exercise against a public website (`redacted.com`), a misconfiguration was identified where the origin server behind AWS CloudFront was directly reachable from the internet.

This allowed potential bypass of CDN protections, exposing the real infrastructure to direct access.

This document details the methodology, tools, findings, impact, and remediation.

---

## 🎯 Objective

Map the real infrastructure behind a website protected by CloudFront and verify whether the origin server was properly isolated from public access.

---

## 🧰 Tools Used

- crt.sh (Certificate Transparency)
- WHOIS / ASN lookup
- subfinder
- curl
- Wappalyzer
- DNS analysis
- Reverse IP / Reverse DNS
- Manual header inspection

---

## 🪜 Step-by-Step Methodology

### 1️⃣ Certificate Transparency Recon (crt.sh)

Using crt.sh, multiple domains and subdomains related to `redacted.com` were discovered.

This provided additional assets to enumerate and correlate infrastructure.

---

### 2️⃣ Host & ASN Identification

One of the discovered IPs resolved to:

- Hosting provider: Brazilian shared hosting company
- ASN belonging to that provider
- Reverse DNS pointing to shared hosting infrastructure

This strongly suggested a legacy/shared hosting origin.

---

### 3️⃣ Technology Fingerprinting

Using Wappalyzer and header inspection:

| Host                 | Technology Identified                         |
|----------------------|-----------------------------------------------|
| redacted.com         | Apache (redirect)                             |
| www.redacted.com     | CloudFront → ALB → Apache/PHP                 |
| webmail.redacted.com | IIS 8.5 on Windows Server (third-party mail)  |

This revealed three distinct infrastructures:

- CDN-protected frontend
- Legacy/shared origin host
- Third-party email service

---

### 4️⃣ Testing Direct Access to the Suspected Origin

A direct request was sent to the suspected origin IP using the Host header:

curl http://X.X.X.X
 -H "Host: redacted.com"

 
**Result**

The server responded with:

- `301 Moved Permanently → https://www.redacted.com`
- `Server: Apache`

This confirms the origin server is publicly reachable and actively serving the website.

---

### 5️⃣ Verifying CloudFront Protection Bypass

Requesting the main site normally:

curl -I https://www.redacted.com


Headers showed:

- `X-Cache: Error from cloudfront`
- `Via: CloudFront`
- AWS ALB cookies

However, direct origin access bypassed all of this.

---

## 🚨 Finding — Origin Server Exposure

### Description

The origin server behind CloudFront was accessible directly via public IP and responded correctly when the Host header was supplied.

This means CloudFront was not acting as the only entry point, breaking a core CDN security model.

---

## 💥 Security Impact

An attacker could:

- Bypass WAF/CDN protections
- Perform direct vulnerability scanning on the origin
- Attempt brute force, fuzzing, or exploitation without CloudFront filtering
- Discover legacy services and outdated software (Apache/PHP on shared hosting)
- Potentially enumerate other virtual hosts on the same server (shared hosting risk)

---

## 🧠 Why This Happens

This is a common misconfiguration when:

- A site migrates to CloudFront
- Old hosting remains publicly accessible
- Firewall/Security Group rules do not restrict origin access to AWS IP ranges only

---

## 🛠️ Recommended Remediation

The origin server should:

- Only accept traffic from CloudFront IP ranges
- Block all public access via firewall or security group
- Be migrated from shared hosting to private infrastructure
- Enforce origin validation (Origin Access Control or custom headers)

---

## 🧩 Additional Observation — Subdomain Segmentation

The `webmail.redacted.com` subdomain was hosted on a different provider running IIS on Windows Server.

This demonstrates:

- Third-party service exposure
- A different attack surface
- Independent infrastructure requiring assessment

---

## 📚 Key Takeaways

This case demonstrates practical skills in:

- Passive reconnaissance
- Infrastructure mapping
- CDN architecture analysis
- Host header testing
- Identifying legacy origin exposure
- Writing professional security reports

---

## 🏁 Conclusion

This assessment identified a critical architectural misconfiguration where a CloudFront-protected website still had its origin server publicly accessible, allowing full bypass of CDN protections.

Such findings are common in real pentest engagements and represent high-value security insights.

---

## ⚖️ Ethical Note

No exploitation was performed.

Only passive reconnaissance and header-based validation were conducted.

This case is shared strictly for educational and professional portfolio purposes.
