# Enterprise Self-Hosted Mail Platform on AWS (Mailcow + SES Smart Relay)

## Project Overview

This project demonstrates the deployment of a production-style self-hosted enterprise email platform on AWS using Mailcow, Postfix, Dovecot, SOGo, Cloudflare DNS, and Amazon SES Smart Host Relay.

The environment was designed to simulate real-world enterprise mail architecture, including:

* SMTP mail delivery
* IMAP mailbox storage
* Webmail access
* SPF/DKIM/DMARC email authentication
* TLS encrypted mail transport
* Outbound smart-host relay architecture
* Inbound MX-based mail routing
* Mail security concepts and troubleshooting

This was not simply a web hosting project — this was a full messaging infrastructure deployment and validation exercise.

---

# High-Level Goals

The main objective of this project was to understand:

* How enterprise email systems actually work
* The difference between inbound vs outbound mail flow
* SMTP relay architecture
* Mail authentication mechanisms
* Smart host design patterns
* DNS mail routing concepts
* Real-world troubleshooting scenarios
* Cloud provider SMTP restrictions

---

# Architecture Overview

## Core Components

| Component         | Purpose                 |
| ----------------- | ----------------------- |
| EC2 Ubuntu Server | Hosted Mailcow platform |
| Mailcow           | Complete mail platform  |
| Postfix           | SMTP mail server        |
| Dovecot           | IMAP/mailbox storage    |
| SOGo              | Webmail interface       |
| Cloudflare        | Authoritative DNS       |
| Amazon SES        | Outbound smart relay    |
| Gmail             | External mail testing   |

---

# Enterprise Mail Flow

## Inbound Mail Flow

```text
Internet Sender
    ↓
MX Record Lookup
    ↓
mail.basitcloudlab.com
    ↓
AWS EC2 / Postfix
    ↓
Dovecot Mailbox Storage
    ↓
SOGo Webmail Inbox
```

## Outbound Mail Flow

```text
SOGo Webmail
    ↓
Postfix SMTP
    ↓
Amazon SES Smart Host Relay
    ↓
Gmail SMTP Infrastructure
    ↓
Recipient Inbox
```

---

# Real-World Enterprise Concepts Learned

## SMTP Relay

A relay server accepts email from one mail server and forwards it onward to another mail server.

In this project:

```text
Postfix
→ Amazon SES
→ Gmail
```

Amazon SES acted as the smart-host outbound SMTP relay.

---

## Smart Host Architecture

A smart host is a trusted outbound SMTP relay used for:

* Deliverability
* Reputation management
* Centralized logging
* TLS enforcement
* Spam prevention
* Cloud SMTP restrictions

This is extremely common in enterprise environments.

---

## Inbound vs Outbound Mail

### Inbound Mail

Controlled by:

```text
MX records
```

Mail flow:

```text
Internet
→ MX lookup
→ Postfix
→ Dovecot
→ SOGo
```

### Outbound Mail

Controlled by:

```text
SMTP relay configuration
```

Mail flow:

```text
SOGo
→ Postfix
→ SES relay
→ Internet
```

---

## Security Gateway Concepts

The project also explored how enterprise email security platforms such as:

* Barracuda
* Proofpoint
* Mimecast
* Cisco ESA

fit into mail architecture.

Example enterprise inbound flow:

```text
Internet
→ Barracuda/Proofpoint
→ Spam & malware inspection
→ Internal Exchange/M365/Mailcow
```

These products typically act as inbound security relays before email reaches internal mailbox servers.

---

# AWS Services Used

| AWS Service     | Purpose                         |
| --------------- | ------------------------------- |
| EC2             | Hosted Mailcow server           |
| Security Groups | Mail traffic filtering          |
| Elastic IP      | Static public mail IP           |
| Amazon SES      | SMTP smart relay                |
| IAM             | SMTP authentication credentials |

---

# DNS Configuration

## Mail Records Configured

| Record Type    | Purpose                     |
| -------------- | --------------------------- |
| A Record       | Mail server hostname        |
| MX Record      | Inbound mail routing        |
| SPF TXT        | Authorized sending servers  |
| DKIM TXT/CNAME | Cryptographic email signing |
| DMARC TXT      | Email authentication policy |

---

# Mail Authentication

## SPF

SPF validates:

```text
Which servers are allowed to send email for the domain.
```

Configured example:

```text
v=spf1 include:amazonses.com -all
```

---

## DKIM

DKIM provides:

```text
Cryptographic signing of outbound email.
```

This helps receiving providers verify email authenticity.

---

## DMARC

DMARC defines:

```text
What receiving mail providers should do when SPF or DKIM fails.
```

Policies tested:

* p=none
* p=quarantine

---

# Major Troubleshooting Scenarios

## 1. AWS SMTP Port 25 Restriction

### Problem

Outbound mail failed with:

```text
Connection timed out
status=deferred
```

### Root Cause

AWS blocked outbound SMTP port 25.

### Resolution

Configured Amazon SES SMTP smart relay on port 587 with TLS.

---

## 2. SES Sandbox Restriction

### Problem

SES rejected mail with:

```text
Email address is not verified
```

### Root Cause

SES accounts begin in sandbox mode.

### Resolution

Verified recipient Gmail identity inside SES.

---

## 3. SPF Validation Issues

### Problem

Incorrect SPF policies caused authentication inconsistencies.

### Resolution

Updated SPF policy to authorize SES relay infrastructure.

---

## 4. DKIM DNS Validation

### Problem

SES domain verification remained pending.

### Root Cause

Missing SES DKIM CNAME records.

### Resolution

Added SES-generated DKIM CNAME records into Cloudflare DNS.

---

## 5. Mail Delivery Reputation Behavior

Observed:

* Spam placement
* Reputation sensitivity
* Authentication dependency
* DNS propagation effects

This demonstrated how modern mail systems evaluate sender trust.

---

# Validation Testing Performed

## Successful Tests

### Outbound Mail Delivery

Validated:

```text
Mailcow
→ SES relay
→ Gmail inbox
```

---

### Inbound Mail Delivery

Validated:

```text
Gmail
→ MX lookup
→ Postfix
→ Dovecot
→ SOGo inbox
```

---

### Authentication Validation

Confirmed:

* SPF PASS
* DKIM PASS
* DMARC PASS

using Gmail message header analysis.

---

### Security / Failure Testing

Tested:

* SPF policy modifications
* DMARC policy changes
* DKIM validation concepts
* Mail reputation behavior
* SMTP relay failure scenarios

---

# Screenshots Captured

Suggested screenshot categories:

* Mailcow admin dashboard
* Mailbox creation
* SOGo webmail login
* SES identity verification
* SES SMTP relay configuration
* Successful Gmail delivery
* SPF/DKIM/DMARC PASS validation
* Inbound mail validation
* Authentication testing
* Troubleshooting scenarios

---

# Key Enterprise Lessons Learned

* SMTP and IMAP are completely different protocols
* Inbound and outbound mail are separate architectures
* MX records only control inbound delivery
* SMTP relays control outbound delivery
* SES smart hosts are commonly used in enterprise environments
* Cloud providers often restrict direct SMTP delivery
* SPF, DKIM, and DMARC work together as layered protection
* Mail reputation is critical for deliverability
* Enterprise email systems are heavily security-focused

---

# Skills Demonstrated

## Cloud & Infrastructure

* AWS EC2
* Elastic IP management
* Security Groups
* Amazon SES
* IAM

## Linux & System Administration

* Ubuntu server administration
* Docker container management
* Service troubleshooting
* SMTP diagnostics
* Log analysis

## Networking & Messaging

* SMTP
* IMAP
* DNS mail routing
* TLS mail encryption
* Smart host relay architecture
* Mail authentication

## Security Concepts

* SPF
* DKIM
* DMARC
* Anti-spoofing controls
* Secure mail transport
* Mail reputation
* Security gateways

---

# Future Improvements

Potential enterprise enhancements:

* Reverse DNS (PTR) production alignment
* Production SES access request
* Spam filtering tuning
* Multi-domain hosting
* High availability mail architecture
* Backup MX servers
* SIEM integration
* Centralized mail logging
* Spam quarantine workflows
* Exchange hybrid integration

---

# Final Outcome

This project successfully demonstrated a fully functional enterprise-style email platform capable of:

* Hosting mailboxes
* Sending authenticated email
* Receiving external email
* Relaying outbound SMTP securely
* Encrypting SMTP traffic
* Performing enterprise-grade mail authentication
* Supporting webmail access
* Integrating with cloud relay infrastructure

The final deployment closely mirrored real-world enterprise messaging architecture patterns used in modern organizations.

---

# Cleanup Performed

The environment was fully decommissioned after testing:

* EC2 terminated
* Elastic IP released
* EBS volumes deleted
* SES identities removed
* SMTP IAM credentials removed
* Mail DNS records removed from Cloudflare

This ensured no unnecessary AWS resource charges remained active.
