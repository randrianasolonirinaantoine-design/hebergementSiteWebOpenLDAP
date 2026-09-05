# 🌐 Enterprise Network Services Architecture — L2ENI Domain Infrastructure
> **Hands-on Systems & Network Engineering Lab — BIND9 DNS, OpenLDAP Directory Services, Apache HTTPS, Custom PHP App, Postfix/Dovecot Mail, Roundcube Webmail & Prometheus/Grafana Monitoring**
## 📌 Project Overview
This project demonstrates the design, deployment, security hardening, and validation of a centralized enterprise-style network infrastructure for the domain **`l2eni.mg`** in a fully virtualized environment using **Debian 12 / Ubuntu Server 22.04 LTS**.
The laboratory environment was designed to reproduce a small-to-medium enterprise (SME) infrastructure where all core network services are interconnected, centrally authenticated, monitored in real time, and secured.
The architecture includes:
- **BIND9** as the primary authoritative DNS server for `l2eni.mg` and reverse DNS (`100.168.192.in-addr.arpa`)
- **OpenLDAP** (`dc=l2eni,dc=mg`) for centralized user identity and authentication
- **Apache2 Web Server** with virtual hosting (`appli.l2eni.mg`, `webmail.l2eni.mg`, `monitoring.l2eni.mg`) and TLS/HTTPS encryption
- **Custom PHP Web Application** (`appli.l2eni.mg`) providing live service monitoring, LDAP directory search, and architecture visualization
- **Postfix & Dovecot Mail Server** for SMTP/IMAP message delivery with LDAP SASL authentication and Maildir storage
- **Roundcube Webmail** (`webmail.l2eni.mg`) providing a responsive browser client for enterprise messaging
- **Prometheus & Grafana Monitoring Stack** (`monitoring.l2eni.mg`) with Alertmanager and Node Exporter for real-time telemetry and alerting
The project follows a complete system engineering workflow:
```text
Design & Planning
  ↓
Deployment
  ↓
Network Setup (192.168.100.10/24)
  ↓
DNS & Identity Provisioning (BIND9 / OpenLDAP)
  ↓
Web & Mail Infrastructure (Apache / Postfix / Dovecot)
  ↓
Monitoring & Alerting Setup (Prometheus / Grafana)
  ↓
Validation & Testing
  ↓
Analysis
  ↓
Hardening
⚠️ Disclaimer: All configurations, authentication tests, and network traffic analyses presented in this repository were conducted exclusively in an authorized and controlled laboratory environment for educational purposes (L2 ENI - École Nationale d'Informatique).

🎯 Objectives
The main objectives of this project were to:

Design and deploy a single-server enterprise services architecture for the l2eni.mg domain.
Configure static networking (192.168.100.10/24) and hostname resolution (srv-l2eni.l2eni.mg).
Deploy BIND9 as a primary authoritative DNS server for forward and reverse lookup zones.
Deploy OpenLDAP to store users, groups, and system accounts in a centralized Directory Information Tree (DIT).
Configure Apache2 VirtualHosts for subdomains (appli, webmail, monitoring) with SSL/TLS bindings.
Develop a custom PHP management interface with live LDAP authentication and system diagnostics.
Configure Postfix (SMTP) and Dovecot (IMAP/LMTP) with LDAP passdb/userdb integration.
Deploy Roundcube Webmail to allow web-based email exchange between LDAP users.
Deploy Prometheus, Node Exporter, Alertmanager, and Grafana for full-stack system metrics and visual dashboard telemetry.
Perform comprehensive protocol validation (dig, ldapsearch, curl, telnet, mailq, prometheus).
Identify configuration weaknesses and propose hardening measures.
🏗️ Network Architecture
The infrastructure is divided into dedicated virtual subdomains and listening ports:

Service / Subdomain	Network Address	Port(s)	Role & Responsibility
Server Host (srv-l2eni.l2eni.mg)	192.168.100.10	N/A	Primary Linux Host (Debian 12 / Ubuntu 22.04)
DNS Server (ns1.l2eni.mg)	192.168.100.10	53 (UDP/TCP)	BIND9 Authoritative & Recursive Resolver
Directory Server (ldap.l2eni.mg)	192.168.100.10	389 (TCP)	OpenLDAP Directory Information Tree
Web Admin App (appli.l2eni.mg)	192.168.100.10	80 / 443 (TCP)	Apache2 VirtualHost + Custom PHP App
Webmail Client (webmail.l2eni.mg)	192.168.100.10	80 / 443 (TCP)	Roundcube Web Interface
Monitoring Dashboard (monitoring.l2eni.mg)	192.168.100.10	3000 (TCP)	Grafana Metrics & Alert Dashboard
Prometheus Server	192.168.100.10	9090 (TCP)	Time-Series Metrics Engine
Alertmanager	192.168.100.10	9093 (TCP)	Infrastructure Alert Dispatcher
Node Exporter	192.168.100.10	9100 (TCP)	OS & Hardware Metrics Exporter
Mail Transport (mail.l2eni.mg)	192.168.100.10	25 / 587 (TCP)	Postfix SMTP & Submission
Mail Storage / IMAP	192.168.100.10	143 / 993 (TCP)	Dovecot IMAP / LMTP Engine
Logical Architecture
text


                        INTERNET / LOCAL LAN
                                 │
                         192.168.100.0/24
                                 │
                       ┌───────────────────┐
                       │   CLIENT PC / UE  │
                       └─────────┬─────────┘
                                 │
                 ┌───────────────┴───────────────┐
                 │ DNS Query: appli.l2eni.mg ?   │
                 ▼                               ▼
       ┌──────────────────┐            ┌──────────────────┐
       │   BIND9 DNS      │            │ OpenLDAP         │
       │   Port 53        │            │ Port 389         │
       └─────────┬────────┘            └─────────┬────────┘
                 │ Resolves IP                   │ Central Auth
                 ▼                               ▼
       ┌──────────────────────────────────────────────────┐
       │             APACHE2 WEB SERVER                   │
       │             Port 80 (HTTP) → 443 (HTTPS)         │
       │  ┌──────────────────┬─────────────────┬───────┐  │
       │  │ appli.l2eni.mg   │ webmail.l2eni.mg│  ...  │  │
       │  │ (Custom PHP UI)  │ (Roundcube)     │       │  │
       │  └────────┬─────────┴────────┬────────┴───┬───┘  │
       └───────────┼──────────────────┼────────────┼──────┘
                   │                  │            │ Reverse Proxy / Port 3000
                   │ ldap_bind()      │ IMAP:993   ▼
                   ▼                  ▼    ┌────────────────┐
           ┌──────────────────────────┐    │ Grafana &      │
           │    POSTFIX + DOVECOT     │    │ Prometheus     │
           │ SMTP (25/587) IMAP (143) │    │ Ports 3000/9090│
           └──────────────────────────┘    └────────────────┘
🔐 Security Policy
The network follows a defense-in-depth approach based on:

Centralized identity & access control
Least privilege
Enforced HTTPS / TLS encryption
Controlled service exposure
Real-time telemetry & alerting
System hardening
Traffic & Service Policy Matrix
Source Service	Target Service	Protocol / Port	Policy	Purpose
Client PC	BIND9 DNS	UDP/TCP 53	✅ ALLOW	Name resolution for *.l2eni.mg
Client PC	Apache Web	TCP 80 / 443	✅ ALLOW (HTTPS Redirect)	Web application & Webmail access
Client PC	Grafana Dashboard	TCP 3000 / 443	✅ ALLOW (Authenticated)	System health & metric monitoring
PHP App	OpenLDAP	TCP 389	✅ ALLOW (Internal Bind)	User authentication & directory query
Roundcube	Dovecot IMAP	TCP 143 / 993	✅ ALLOW	Reading user mailboxes via IMAP
Roundcube	Postfix SMTP	TCP 587 (Submission)	✅ ALLOW (SASL Auth)	Sending email messages
Prometheus	Node Exporter	TCP 9100	✅ ALLOW (Local Scrape)	Host CPU/RAM/Disk metrics scraping
Prometheus	Alertmanager	TCP 9093	✅ ALLOW (Internal)	Routing threshold alerts
🌐 BIND9 DNS Server
BIND9 acts as the primary authoritative DNS server for l2eni.mg.

It is responsible for:

Forward domain lookup (l2eni.mg → 192.168.100.10)
Reverse DNS lookup (10.100.168.192.in-addr.arpa → srv-l2eni.l2eni.mg)
MX record routing (mail.l2eni.mg)
Subdomain resolution (appli, webmail, monitoring)
Key Records Defined
text


@           IN  SOA     srv-l2eni.l2eni.mg. admin.l2eni.mg.
@           IN  NS      srv-l2eni.l2eni.mg.
srv-l2eni   IN  A       192.168.100.10
appli       IN  A       192.168.100.10
webmail     IN  A       192.168.100.10
monitoring  IN  A       192.168.100.10
mail        IN  A       192.168.100.10
@           IN  MX  10  srv-l2eni.l2eni.mg.
🔍 BIND9 DNS Resolution Test
The following screenshot provides evidence of the DNS resolution test performed in the laboratory using dig.

BIND9 DNS Dig Test

👥 OpenLDAP Directory Services
OpenLDAP (slapd) maintains the enterprise Directory Information Tree (DIT).

Directory Structure (DIT)
text


dc=l2eni,dc=mg (Base DN)
 ├── ou=users (Organizational Unit for accounts)
 │    ├── uid=alice (inetOrgPerson, posixAccount)
 │    └── uid=bob   (inetOrgPerson, posixAccount)
 └── ou=groups (Organizational Unit for security groups)
      └── cn=etudiants (posixGroup)
🔑 OpenLDAP Search & Authentication Test
The following screenshots provide evidence of directory schema search and single-user authentication using ldapsearch and ldapwhoami.

OpenLDAP Directory Search

👤 OpenLDAP User Authentication Test
OpenLDAP User Authentication Test

🌐 Web Server & SSL/TLS Encryption
Apache2 hosts all subdomains using VirtualHosts and SSL/TLS certificate bindings (/etc/ssl/l2eni/l2eni.crt).

Implemented Web Services
appli.l2eni.mg → Custom PHP Web Application (/var/www/appli/)
webmail.l2eni.mg → Roundcube Webmail (/var/www/roundcube/)
monitoring.l2eni.mg → Grafana Reverse Proxy (http://127.0.0.1:3000/)
🔒 SSL Certificate Details
The screenshot below validates the HTTPS SSL binding and certificate details.

SSL Certificate Details

🔐 Custom App Login Page
The screenshot below displays the modern LDAP login page for appli.l2eni.mg.

Custom App Login Page

📊 Custom App Dashboard
The screenshot below displays the admin dashboard after successful LDAP authentication.

Custom App Dashboard

👥 LDAP User Management Page
The screenshot below displays the real-time LDAP directory user search page.

LDAP User Directory Page

🗺️ Network Architecture Documentation Page
The screenshot below displays the visual network architecture documentation page.

Architecture Diagram Page

📮 Enterprise Mail Infrastructure
The mail system provides end-to-end messaging:

Postfix (MTA): Handles SMTP incoming mail (Port 25) and authenticated submission (Port 587).
Dovecot (MDA/IMAP): Delivers mail via LMTP into /var/mail/vhosts/l2eni.mg/%n and serves IMAP/IMAPS.
Roundcube (Webmail): Offers a web GUI with LDAP addressbook integration.
⚡ Mail Service Status & Listening Ports
The screenshot below validates active system listening ports (25, 587, 143, 993, 389, 443, 53).

Service Status & Listening Ports

📮 Roundcube Webmail Login
The screenshot below displays the webmail login page for webmail.l2eni.mg.

Roundcube Webmail Login

✉️ Webmail Email Exchange Test
The screenshot below displays an end-to-end email exchange test between alice and bob.

Webmail Email Exchange

📊 Prometheus & Grafana Monitoring
A dedicated real-time monitoring and alerting stack was deployed to observe server health and service metrics:

Node Exporter (:9100): Collects hardware/OS metrics (CPU load, memory consumption, disk I/O, network bandwidth).
Prometheus (:9090): Time-series database scraping metrics targets every 15 seconds.
Alertmanager (:9093): Evaluates firing alerts and routes threshold notifications.
Grafana (:3000 / monitoring.l2eni.mg): Web dashboard presenting visual telemetry graphs and alert status panels.
📈 Grafana Monitoring & Telemetry Dashboard
The screenshot below displays the Grafana authentication and telemetry monitoring interface.

Grafana Monitoring Login & Telemetry

📊 Security & Service Assessment Results
Security & Service Control	Result	Assessment
Network IP Assignment	✅ PASS	Static IP 192.168.100.10/24 configured
Forward DNS Resolution	✅ PASS	Authoritative forward lookup functional
Reverse DNS PTR	✅ PASS	Reverse PTR resolution functional
OpenLDAP Bind Auth	✅ PASS	Centralized authentication verified
HTTP → HTTPS Redirect	✅ PASS	Port 80 redirected to 443
SSL/TLS Encryption	✅ PASS	SSL VirtualHost active
Custom PHP App Auth	✅ PASS	Session created from LDAP
Postfix SMTP Submission	✅ PASS	SASL authentication enforced
Dovecot IMAP Delivery	✅ PASS	Maildir local delivery verified
Webmail Email Exchange	✅ PASS	Full messaging workflow validated
Prometheus Metrics Scraping	✅ PASS	Time-series metrics collection functional
Grafana Dashboard Telemetry	✅ PASS	Visual monitoring dashboard active
Self-Signed SSL Certificate	⚠️ NEEDS IMPROVEMENT	Production requires CA-signed certificate
Unencrypted LDAP (Port 389)	⚠️ WEAK	Enforce STARTTLS or LDAPS (Port 636)
⚠️ Identified Weaknesses
The assessment identified several areas requiring improvement before production deployment.

1. Self-Signed SSL Certificate
The current TLS certificate is self-signed (/etc/ssl/l2eni/l2eni.crt), causing browser warnings.

Recommendation
Deploy Let's Encrypt (Certbot) for public domains.
Establish an internal Root Certificate Authority (CA) for private laboratory networks.
2. Unencrypted LDAP Traffic (Port 389)
LDAP traffic currently flows over standard unencrypted TCP 389 on localhost.

Recommendation
Enable LDAPS (Port 636) or enforce STARTTLS on port 389 to protect directory credentials in transit.

3. Single Point of Failure (SPOF)
All services (DNS, LDAP, Web, Mail, Monitoring) reside on a single virtual host instance.

Recommendation
Separate roles into dedicated virtual machines or containers (e.g., DNS VM, LDAP VM, Web DMZ VM, Monitoring VM).

🔧 Hardening Recommendations
OpenLDAP & Directory
Enforce strong password complexity rules via slapd-ppolicy.
Restrict anonymous bind access explicitly in slapd.conf / cn=config.
Implement automated off-site LDIF backups using slapcat.
Web Server (Apache)
Enable HTTP Security Headers in VirtualHosts:
text


Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set Content-Security-Policy "default-src 'self';"
Disable server signature tokens (ServerTokens Minimal, ServerSignature Off).
Mail Server (Postfix / Dovecot)
Enable DNSBL (DNS Blocklists) in Postfix to filter spam: smtpd_recipient_restrictions = reject_rbl_client zen.spamhaus.org.
Configure DKIM (OpenDKIM) and DMARC DNS records for outbound email validation.
Monitoring & Alerting (Grafana / Prometheus)
Restrict Prometheus (:9090) and Node Exporter (:9100) to localhost or internal monitoring subnets only.
Enforce HTTPS for Grafana (monitoring.l2eni.mg) and enable OAuth2 / LDAP authentication for Grafana admin users.
🧪 Testing Methodology
The laboratory follows a simplified system integration life cycle:

text


1. Network & Static IP Setup (192.168.100.10)
          ↓
2. BIND9 DNS Zone Provisioning
          ↓
3. OpenLDAP Schema & User Seeding
          ↓
4. Apache VirtualHost & SSL Certificate Configuration
          ↓
5. Custom PHP Web App Deployment
          ↓
6. Postfix & Dovecot LDAP Integration
          ↓
7. Roundcube Webmail Setup
          ↓
8. Prometheus & Grafana Monitoring Deployment
          ↓
9. End-to-End Functional Testing
          ↓
10. Diagnostic Analysis & Hardening
🧠 Skills Demonstrated
Networking & Protocols
IPv4 Subnetting & Static Routing (192.168.100.10/24)
DNS Zone File Syntax (SOA, NS, A, MX, PTR, TXT, SPF)
HTTP/HTTPS Virtual Host Routing & Reverse Proxying
SMTP / IMAP / LMTP Protocol Flow & Ports
System Administration
OpenLDAP DIT Design, LDIF Scripting & slapd Reconfiguration
Linux User & Group Permissions (vmail, www-data, chown/chmod)
Apache2 Module Management (mod_ssl, mod_rewrite, mod_proxy)
Postfix main.cf / master.cf & Dovecot auth-ldap Integration
Defensive Security & Observability
System Hardening & Access Control
Prometheus Time-Series Metrics Scraping & PromQL Querying
Node Exporter Telemetry Collection (CPU, Memory, Disk, Network)
Alertmanager Rule Thresholding & Routing
Grafana Dashboard Design & Visual Analytics
Web & Application Development
Modern PHP LDAP Integration (ldap_connect, ldap_bind, ldap_search)
Session State Management & Security
Vanilla CSS Design System (Glassmorphism & Responsive Layout)
Webmail Integration (Roundcube Database & Configuration)
🛠️ Technology Stack
Category	Technology
Operating System	Debian 12 / Ubuntu Server 22.04 LTS
DNS Server	BIND9 (named)
Directory Server	OpenLDAP (slapd)
Web Server	Apache2
Programming Language	PHP 8.2 (with php-ldap, php-curl, php-mbstring)
Mail Transfer Agent	Postfix
Mail Delivery Agent	Dovecot
Webmail Client	Roundcube 1.6.6
Metrics Engine	Prometheus 2.x
Metrics Exporter	Node Exporter
Alerting System	Alertmanager
Visualization UI	Grafana 10.x
Database	MariaDB (for Roundcube & Grafana)
Security & SSL	OpenSSL (X.509 Certificates)
📸 Evidence & Validation
The following screenshots document the practical implementation and validation of the laboratory.

Network & Infrastructure Setup
Static IP & Network Interface
Static IP & Netplan Configuration

BIND9 DNS Resolution Test
BIND9 DNS Dig Test

OpenLDAP Directory Services
LDAP Directory Search
OpenLDAP Directory Search

User Authentication Test
User Authentication Test

Web Services & Custom Application
SSL/TLS Certificate Verification
SSL Certificate Details

Custom App Login Page
Custom App Login Page

Admin Dashboard
Admin Dashboard

LDAP User Management
LDAP User Directory

Architecture Documentation Page
Architecture Diagram Page

Messaging & Webmail Services
Mail Service Status & Listening Ports
Service Status & Listening Ports

Roundcube Webmail Login
Roundcube Webmail Login

Full Mail Exchange Test
Webmail Email Exchange

Infrastructure Monitoring
Grafana Monitoring Dashboard
Grafana Monitoring Login & Telemetry

📈 Infrastructure & Security Summary
The architecture demonstrates how multiple network services interact in a unified enterprise environment:

text


                        CENTRALIZED NETWORK INFRASTRUCTURE
                                  BIND9 DNS
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
           Apache2 Web            OpenLDAP             Prometheus /
        (appli / webmail)      (Central Auth)           Grafana
                 │                    │                    │
                 └────────────────────┼────────────────────┘
                                      │
                             Postfix / Dovecot
                                (SMTP / IMAP)
                                      │
                                      ▼
                             Validated Delivery
🛡️ Defense in Depth
The infrastructure architecture follows a layered security model:

text


                 ┌──────────────────┐
                 │ Static IP Setup  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ BIND9 DNS        │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ OpenLDAP Auth    │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Apache HTTPS     │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Postfix / Dovecot│
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Prometheus/Grafana│
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Hardening        │
                 └────────┬─────────┘
This layered approach ensures that identity, encryption, messaging, and telemetry work together to protect and monitor the infrastructure.

🚀 Future Improvements
Possible future developments include:

Deploying OpenLDAP Replication (Syncrepl) for high availability.
Configuring Let's Encrypt / Certbot for automated SSL certificate renewal.
Integrating Wazuh SIEM / Fail2ban for brute-force protection on SSH, Web, and Mail ports.
Separating services into isolated Docker containers or KVM virtual machines.
Implementing DKIM, DMARC, and DNSSEC for strict email and domain validation.
Configuring automated Slack/Discord/Email notifications in Alertmanager.
⚠️ Laboratory Limitations
This project is an educational engineering laboratory and should not be considered a production-ready enterprise architecture without additional controls.

A production environment would require:

High-availability redundant DNS & LDAP servers
VLAN-based network segmentation
Centralized SIEM and EDR agent monitoring
Multi-factor authentication (MFA)
Enterprise Public Key Infrastructure (PKI)
Automated off-site backups and disaster recovery plans
🏁 Conclusion
This project demonstrates the practical design, implementation, and validation of a centralized network infrastructure using:

BIND9 + OpenLDAP + Apache2 + Custom PHP App + Postfix + Dovecot + Roundcube + Prometheus + Grafana

The laboratory validates the complete system lifecycle:

text


DESIGN → DEPLOY → PROVISION → INTEGRATE → MONITOR → TEST → ANALYZE → HARDEN
The main lesson demonstrated by this laboratory is that effective system administration requires multiple complementary infrastructure layers working in unison:

text


Static Networking
      +
Authoritative DNS
      +
Directory Identity (OpenLDAP)
      +
HTTPS Web Services
      +
Authenticated Messaging
      +
Real-time Monitoring (Prometheus/Grafana)
      =
Enterprise Infrastructure Readiness
📚 Project Information
Item	Details
Project Type	Enterprise Network Services & Monitoring Lab
Institution	École Nationale d'Informatique (ENI)
Academic Level	L2 Network & Systems Engineering
Domain Name	l2eni.mg
Primary IP	192.168.100.10/24
Target OS	Debian 12 / Ubuntu Server 22.04 LTS
Repository License	MIT
All network and service tests were performed in an isolated and authorized virtual laboratory environment for academic and practical engineering validation.
