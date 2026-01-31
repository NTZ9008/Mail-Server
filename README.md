📧 Ultimate Home Lab Mail Server (Hybrid Cloud-Premise)
"Transforming a residential Mini PC into an Enterprise-grade Mail Server, bypassing ISP restrictions via secure Cloud Tunneling."

📖 Overview
This project demonstrates how to deploy a fully functional Self-Hosted Mail Server (Mailcow) on residential hardware (Mini PC) despite common ISP limitations such as Dynamic IPs, CGNAT, and Port 25 blocking.
By utilizing a Hybrid Cloud Architecture, an Oracle Cloud VPS acts as a public-facing gateway, tunneling traffic securely to the home server via Tailscale. The system preserves Real Client IPs for accurate security logging and employs a Smart Relay for 100% email deliverability.

🏗 Network Topology
The architecture uses a Bi-directional Proxy approach, separating inbound and outbound traffic flows for maximum security and compatibility.

graph TD
    %% Actors
    Sender((📩 External Sender))
    User((📱 Me/Client App))
    Recipient((📬 Recipient Inbox))

    %% Cloud Segment
    subgraph Cloud [☁️ Oracle Cloud VPS (Gateway)]
        Firewall[🔥 Cloud Firewall]
        Nginx[⚙️ Nginx Stream Proxy]
    end

    %% Tunnel
    subgraph Tunnel [🔒 Tailscale VPN]
        Link[<== Encrypted WireGuard Tunnel ==>]
    end

    %% Home Segment
    subgraph Home [🏠 Home Mini PC]
        Postfix[📮 Postfix / Mailcow]
        Dovecot[📂 Dovecot (IMAP)]
    end

    %% External Services
    Brevo[🚀 Brevo SMTP Relay]

    %% Flows
    %% 1. Inbound Mail (Port 25)
    Sender -- "SMTP (25)" --> Firewall
    Firewall --> Nginx
    Nginx -- "Proxy Protocol (Real IP)" --> Link
    Link --> Postfix

    %% 2. Client Access (Read/Send)
    User -- "IMAP (993) / SUBMISSION (587)" --> Firewall
    Firewall --> Nginx
    Nginx -- "Direct Stream" --> Link
    Link --> Dovecot

    %% 3. Outbound Mail
    Postfix -- "Smart Relay" --> Brevo
    Brevo -- "Clean IP Delivery" --> Recipient


🚀 Key Features

🛡️ CGNAT & ISP Bypass: Overcame the lack of a public IP and blocked Port 25 by tunneling traffic through a Cloud VPS.
🕵️ Real IP Restoration: Implemented PROXY Protocol on Nginx (VPS) and Postfix (Home). The home server sees the original sender's IP, not the VPN IP. This is critical for:
SPF Checks: Validating sender identity.
Fail2Ban: Banning attackers accurately without locking out the gateway.

📧 High Deliverability: Outbound emails are routed via Brevo SMTP Relay (Port 587) to ensure they land in the Inbox (Gmail, Outlook) and avoid residential IP blacklists.

🔒 Secure Tunneling: All traffic between Cloud and Home is encrypted via WireGuard (Tailscale).

⚙️ Technical Implementation

1. VPS Gateway Configuration (Nginx Stream)
Replaced iptables NAT with Nginx Stream to handle traffic routing.
Port 25: Enables proxy_protocol to pass the Real IP to Postfix.
Port 587/993: Disables proxy_protocol to ensure compatibility with standard email clients (Outlook/Mobile).

# /etc/nginx/nginx.conf (Snippet)
stream {
    # Port 25 (Incoming SMTP): Proxy Protocol ON for Real IP/SPF Checks
    server {
        listen 25;
        proxy_pass 100.x.x.x:25; # Tailscale IP
        proxy_protocol on; 
    }

    # Port 587 (Submission) & 993 (IMAP): Proxy Protocol OFF for Client Compatibility
    server {
        listen 587;
        proxy_pass 100.x.x.x:587;
    }
    server {
        listen 993;
        proxy_pass 100.x.x.x:993;
    }
}


2. Postfix Configuration (Home Server)
Configured Postfix to accept HAProxy protocol only from the trusted VPN gateway for incoming connections.
# data/conf/postfix/extra.cf
# Enable Proxy Protocol for Postscreen (Spam filter) to see Real IPs
postscreen_upstream_proxy_protocol = haproxy
# Note: smtpd_upstream_proxy_protocol is explicitly REMOVED 
# to allow authenticated clients (Phone/Webmail) to connect without timeout errors.
