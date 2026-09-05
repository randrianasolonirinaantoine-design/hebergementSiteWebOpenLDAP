🌐 Enterprise Network Services Architecture — L2ENI Domain Infrastructure
Hands-on Systems & Network Engineering Lab — BIND9 DNS, OpenLDAP Directory Services, Apache HTTPS, Custom PHP App, Postfix/Dovecot Mail, Roundcube Webmail & Prometheus/Grafana Monitoring

📌 Project Overview
This project demonstrates the end-to-end design, deployment, security hardening, and empirical validation of a centralized enterprise network infrastructure for the domain l2eni.mg in a Debian 12 / Ubuntu Server environment.

The laboratory environment reproduces a complete small-to-medium enterprise (SME) infrastructure where all core network services are interconnected, centrally authenticated, monitored in real time, and secured.

The architecture includes:

BIND9 DNS Server for primary domain name resolution (l2eni.mg) and reverse DNS (100.168.192.in-addr.arpa)
OpenLDAP Directory Services (dc=l2eni,dc=mg) for centralized user identity and authentication
Apache2 Web Server with virtual hosting (appli.l2eni.mg, webmail.l2eni.mg & monitoring.l2eni.mg) and TLS/HTTPS encryption
Modern PHP Web Application (appli.l2eni.mg) providing live service monitoring, LDAP directory search, and architecture visualization
Postfix & Dovecot Mail Server for SMTP/IMAP message delivery with LDAP SASL authentication and Maildir storage
Roundcube Webmail (webmail.l2eni.mg) providing a responsive web browser client for enterprise messaging
Prometheus & Grafana Monitoring Stack (monitoring.l2eni.mg) with Alertmanager for real-time metrics collection, CPU/RAM/Network telemetry, and automated alerting
The project follows a complete system engineering workflow:

text

Requirements & Planning
          ↓
Network & Host Setup (192.168.100.10/24)
          ↓
DNS Infrastructure (BIND9)
          ↓
Identity & Directory Services (OpenLDAP)
          ↓
Web Services & SSL/TLS Encryption (Apache2)
          ↓
Custom Admin Application (PHP)
          ↓
Mail Transport & Storage (Postfix/Dovecot)
          ↓
Webmail Client Integration (Roundcube)
          ↓
Monitoring & Alerting Stack (Prometheus + Grafana)
          ↓
Validation, Testing & Hardening
⚠️ Disclaimer: All configurations, authentication tests, and network traffic analyses presented in this repository were conducted exclusively in an authorized and controlled laboratory environment for educational purposes (L2 ENI - École Nationale d'Informatique).

🎯 Objectives
The main objectives of this project were to:

Design and deploy a single-server enterprise services architecture for the l2eni.mg domain.
Implement static networking (192.168.100.10/24) and hostname resolution (srv-l2eni.l2eni.mg).
Configure BIND9 as a primary authoritative DNS server for forward and reverse lookup zones.
Deploy OpenLDAP to store users, groups, and system accounts in a centralized Directory Information Tree (DIT).
Configure Apache2 VirtualHosts for subdomains (appli, webmail, monitoring) with SSL/TLS certificate binding.
Develop a custom PHP management interface with live LDAP authentication and system diagnostics.
Configure Postfix (SMTP) and Dovecot (IMAP/LMTP) with LDAP passdb/userdb integration.
Deploy Roundcube Webmail to allow web-based email exchange between LDAP users.
Deploy Prometheus, Node Exporter, Alertmanager, and Grafana for full-stack system metrics and visual dashboard telemetry.
Perform comprehensive protocol validation (dig, ldapsearch, curl, telnet, mailq, prometheus).
Identify configuration weaknesses and propose hardening recommendations.
🏗️ Network & Server Architecture
The infrastructure is centralized on a primary enterprise server instance serving multiple virtual services:

Service / Subdomain	Network Address	Port(s)	Role & Responsibility
Server Host (srv-l2eni.l2eni.mg)	192.168.100.10	N/A	Primary Linux Host (Debian 12 / Ubuntu 22.04)
DNS Server (ns1.l2eni.mg)	192.168.100.10	53 (UDP/TCP)	BIND9 Authoritative & Recursive Resolver
Directory Server (ldap.l2eni.mg)	192.168.100.10	389 (TCP)	OpenLDAP Directory Information Tree
Web Admin App (appli.l2eni.mg)	192.168.100.10	80 / 443 (TCP)	Apache2 VirtualHost + PHP Dashboard
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
🔐 Security Policy & Access Matrix
The infrastructure follows a centralized authentication, monitoring & least-privilege model:

All user credentials are stored exclusively inside OpenLDAP (ou=users,dc=l2eni,dc=mg).
Unencrypted HTTP traffic on Port 80 is permanently redirected to HTTPS (Port 443).
Plaintext authentication is prohibited across external interfaces; TLS/SSL is enforced for Web (HTTPS) and Mail (STARTTLS/SMTPS/IMAPS).
Grafana dashboard access (monitoring.l2eni.mg) is secured via administrator credentials and role-based access control.
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
🌍 DNS Service Architecture — BIND9
BIND9 acts as the primary authoritative DNS server for l2eni.mg.

Configured Zones
Forward Lookup Zone: /etc/bind/zones/db.l2eni.mg (l2eni.mg → 192.168.100.10)
Reverse Lookup Zone: /etc/bind/zones/db.192.168.100 (10.100.168.192.in-addr.arpa → srv-l2eni.l2eni.mg)
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
🔍 BIND9 DNS Validation Test
The following evidence proves valid forward and MX record resolution using dig.

BIND9 DNS Resolution Test

👥 Centralized Directory Services — OpenLDAP
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
The screenshots below validate directory schema search and single-user authentication using ldapwhoami.

OpenLDAP Directory Search

OpenLDAP User Authentication Test

🌐 Web Application & VirtualHosts — Apache2 + HTTPS
Apache2 hosts all subdomains using VirtualHosts and SSL/TLS certificate bindings (/etc/ssl/l2eni/l2eni.crt).

VirtualHost Specifications
Application Root: /var/www/appli/ (appli.l2eni.mg)
Webmail Root: /var/www/roundcube/ (webmail.l2eni.mg)
Monitoring Proxy: ProxyPass / http://127.0.0.1:3000/ (monitoring.l2eni.mg)
HTTP/HTTPS Handling: Automatic 301 redirect from Port 80 to Port 443.
🔐 SSL/TLS & Web Application Evidence
The screenshots below demonstrate HTTPS SSL binding, the modern LDAP login page, the admin dashboard, and the user directory page.

SSL Certificate Details

Web Application Login Page

Web Application Dashboard

LDAP User Directory Page

Architecture Documentation Page

📮 Enterprise Mail Infrastructure — Postfix + Dovecot + Roundcube
The mail system provides end-to-end messaging:

Postfix (MTA): Handles SMTP incoming mail (Port 25) and authenticated submission (Port 587).
Dovecot (MDA/IMAP): Delivers mail via LMTP into /var/mail/vhosts/l2eni.mg/%n and serves IMAP/IMAPS.
Roundcube (Webmail): Offers a complete web GUI with LDAP addressbook integration.
Mail Delivery Workflow
text

Sender (Alice) → Roundcube → Postfix (SMTP:587 + SASL LDAP)
                                    ↓
                            Dovecot LMTP Socket
                                    ↓
                       Maildir (/var/mail/vhosts/l2eni.mg/bob/)
                                    ↓
Receiver (Bob) ← Roundcube ← Dovecot (IMAP:993 + LDAP passdb)
✉️ Mail Service Status & Roundcube Validation
The screenshots below validate active system listening ports and an end-to-end email exchange between users alice and bob.

Mail System Listening Ports

Roundcube Webmail Login

Roundcube Mail Exchange

📊 Infrastructure Monitoring — Prometheus, Grafana & Alertmanager
A dedicated real-time monitoring and alerting stack was deployed to monitor server metrics and service availability:

Node Exporter (:9100): Collects hardware/OS metrics (CPU load, memory consumption, disk I/O, network bandwidth).
Prometheus (:9090): Time-series database that scrapes Node Exporter and service targets every 15 seconds.
Alertmanager (:9093): Evaluates firing alerts (e.g. Service Down, Disk Full, High CPU) and dispatches notifications.
Grafana (:3000 / monitoring.l2eni.mg): Rich web dashboard presenting visual graphs, gauges, and alert status panels.
Monitoring & Alerting Flow
text

Host System / Services
          │
    Node Exporter (9100)
          │ Scrape (15s)
          ▼
    Prometheus Server (9090) ───► Alertmanager (9093) ──► Email/Webhook Alert
          │ PromQL Queries
          ▼
    Grafana Dashboard (3000 / monitoring.l2eni.mg)
📈 Monitoring Stack Evidence
The screenshot below demonstrates the Grafana authentication and telemetry monitoring interface.

Grafana Monitoring Login & Dashboard

📊 Service Validation & Test Results
Service Component	Test Executed	Expected Result	Status	Assessment
Network IP	ip a / netplan	IP 192.168.100.10/24 assigned	✅ PASS	Static IP routing established
DNS Resolution	dig @127.0.0.1 appli.l2eni.mg	Returns 192.168.100.10	✅ PASS	Authoritative forward DNS functional
DNS Reverse PTR	dig -x 192.168.100.10	Returns srv-l2eni.l2eni.mg	✅ PASS	Reverse DNS lookup functional
LDAP Bind	ldapwhoami -D "uid=alice..."	dn:uid=alice... returned	✅ PASS	Centralized authentication verified
HTTP Redirect	curl -I http://appli.l2eni.mg	301 Moved Permanently	✅ PASS	Port 80 redirected to 443
HTTPS Security	curl -k -I https://appli.l2eni.mg	HTTP/1.1 200 OK	✅ PASS	SSL/TLS VirtualHost active
PHP App Auth	Web form submission	Redirect to dashboard.php	✅ PASS	Session created from LDAP
SMTP Submission	Port 587 STARTTLS	Authentication required	✅ PASS	SASL prevents open relay
IMAP Delivery	telnet localhost 143	Login OK & mailbox list	✅ PASS	Dovecot Maildir delivery verified
Webmail Flow	Alice → Bob email	Message visible in Inbox	✅ PASS	Full messaging workflow validated
Prometheus Metrics	curl http://localhost:9090	Prometheus UI active	✅ PASS	Time-series scraping functional
Grafana Dashboard	curl http://localhost:3000	Grafana Login 200 OK	✅ PASS	Visualization dashboard active
⚠️ Identified Weaknesses
The security assessment identified several areas requiring improvement before production deployment:

1. Self-Signed SSL Certificate
The current TLS certificate is self-signed (/etc/ssl/l2eni/l2eni.crt), causing browser warnings.

Recommendation
Deploy Let's Encrypt (Certbot) for public domains.
Establish an internal Root Certificate Authority (CA) for private laboratory networks.
2. Unencrypted LDAP Traffic (Port 389)
LDAP traffic currently flows over standard unencrypted TCP 389 on localhost.

Recommendation
Enable LDAPS (Port 636) or enforce STARTTLS on port 389 to protect directory credentials in transit across external subnets.
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
The laboratory follows a rigorous system integration life cycle:

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
System & Directory Administration
OpenLDAP DIT Design, LDIF Scripting & slapd Reconfiguration
Linux User & Group Permissions (vmail, www-data, chown/chmod)
Apache2 Module Management (mod_ssl, mod_rewrite, mod_proxy)
Postfix main.cf / master.cf & Dovecot auth-ldap Integration
Monitoring & Observability
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
Category	Technology / Tool
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
📸 Evidence & Validation Gallery
The following screenshots document the practical implementation and validation of the laboratory:

Network & Infrastructure Setup
1. Static IP & Network Interface (01_netplan_ip.png)
Static IP & Netplan Configuration

2. BIND9 DNS Resolution Test (02_bind9_dig_test.png)
BIND9 DNS Dig Test

OpenLDAP Directory Services
3. LDAP Directory Search (03_ldap_search.png)
OpenLDAP Directory Search

4. User Authentication Test (04_ldap_user_alice.png)
User Authentication Test

Web Services & Custom Application
5. SSL/TLS Certificate Verification (05_ssl_certificate.png)
SSL Certificate Details

6. Custom App Login Page (06_app_login_page.png)
Custom App Login Page

7. Admin Dashboard (07_app_dashboard.png)
Admin Dashboard

8. LDAP User Management (08_app_users_ldap.png)
LDAP User Directory

9. Network Architecture Overview (09_app_architecture.png)
Architecture Diagram Page

Messaging & Webmail Services
10. Mail & Network Service Ports (10_postfix_dovecot_status.png)
Service Status & Listening Ports

11. Roundcube Webmail Login (11_roundcube_login.png)
Roundcube Webmail Login

12. Full Mail Exchange Test (12_roundcube_mail_exchange.png)
Webmail Email Exchange

Monitoring & Infrastructure Telemetry
13. Grafana Monitoring Dashboard (13_monitoring.png)
Grafana Monitoring Login & Telemetry

📈 Infrastructure Summary & Defense-in-Depth
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
🚀 Future Improvements
Implement OpenLDAP Replication (Syncrepl) for high availability.
Configure Let's Encrypt / Certbot for automated SSL certificate renewal.
Integrate Wazuh SIEM / Fail2ban for brute-force protection on SSH, Web, and Mail ports.
Separate services into isolated Docker containers or KVM virtual machines.
Implement DKIM, DMARC, and DNSSEC for strict email and domain validation.
Configure automated Slack/Discord/Email notifications in Alertmanager for server alerts.
🏁 Conclusion
This project demonstrates the practical design, implementation, and validation of an enterprise network infrastructure for l2eni.mg:

BIND9 + OpenLDAP + Apache2 + Custom PHP App + Postfix + Dovecot + Roundcube + Prometheus + Grafana

The laboratory validates core system engineering workflows:

text

DESIGN → PROVISION → INTEGRATE → AUTHENTICATE → MONITOR → SECURE → VALIDATE
The primary takeaway from this project is that centralized identity management (OpenLDAP) combined with authoritative DNS, SSL/TLS web/mail protocols, and real-time observability (Prometheus/Grafana) provides a robust, scalable, and manageable foundation for modern enterprise network services.

📚 Project Metadata
Item	Details
Project Title	L2ENI Enterprise Network Services & Monitoring Lab
Institution	École Nationale d'Informatique (ENI)
Academic Level	L2 Network & Systems Engineering
Domain Name	l2eni.mg
Primary IP	192.168.100.10/24
Target OS	Debian 12 / Ubuntu Server 22.04 LTS
Repository License	MIT
All network and service tests were performed in an isolated, authorized virtual laboratory environment for academic and practical engineering validation.
