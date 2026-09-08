## Homelab DNS and Reverse Proxy Architecture Guide
This document explains how to set up an enterprise-grade internal routing system using wildcard subdomains. This method allows you to access all your services via friendly names (e.g., dns.homelab.lan, vault.homelab.lan) without ever modifying the internal configurations or environment variables of your individual application containers.
## Infrastructure Overview

* Technitium DNS Server: 10.100.10.2 (Web GUI Port: 5380)
* Nginx Reverse Proxy Host: 10.100.10.3 (Web GUI/Admin Port: 81, HTTP Port: 80, HTTPS Port: 443)
* Target Local Domain Zone: homelab.lan (Substitute with your actual local domain)

------------------------------
## Architectural Deep Dive: Subdomains (dns.domain) vs. Path Subfolders (domain/dns)
When designing your networking landscape, navigating to your services can be approached in two distinct ways. Understanding why the enterprise standard strictly dictates using subdomains over subfolders is critical to long-term homelab stability.
## The Subdomain Approach: dns.homelab.lan (Recommended)
In this layout, each application is assigned its own unique subdomain prefix that sits to the left of your primary domain.

* How Web Browsers Handle It: The web browser views dns.homelab.lan and vault.homelab.lan as completely separate, independent websites.
* The Root Path Advantage: Because the hostname itself changes, the application can serve its traffic out of its native root path (/). When Technitium tells your browser to look for its design sheets at /css/style.css, the browser successfully requests dns.homelab.lan/css/style.css.
* Container Configuration Impact: Zero modifications. Because the application remains at the root directory level, you do not have to touch a single environment variable or internal configuration file inside your container deployments. Everything works perfectly right out of the box.

## The Subfolder Approach: homelab.lan/dns (Discouraged)
In this layout, you use a single root domain name and append a folder path to the end of the URL to differentiate your services.

* Why Web Pages Break: Most modern self-hosted applications are hardcoded by their developers to assume they own the absolute root (/) of whatever address they are given. When you map Nginx to listen for homelab.lan/dns, Technitium is unaware it has been forced into a subdirectory. It still instructs your browser to fetch its companion files from the root domain (homelab.lan/css/style.css instead of homelab.lan/dns/css/style.css).
* The Resulting Failure: Nginx checks its configuration for a rule covering /css/, finds nothing, and returns a 404 Not Found error. The browser fails to download the javascript or styles, leaving you with a broken, entirely blank white web page.
* The Security Flaw: Web browsers isolate security tokens, cookies, and local storage variables by domain name. If you utilize homelab.lan/dns and homelab.lan/vault, both systems share the exact same domain space. An exploit or malicious dependency inside one minor container could theoretically gain access to the secure browser sessions or vault storage keys of your password manager.

------------------------------
## Step 1: Technitium DNS Configuration
Instead of mapping every single container manually, you configure a wildcard record. This forces Technitium to send all sub-domain queries to your Nginx proxy machine.

   1. Log into your Technitium DNS console at http://10.100.10.2:5380.
   2. Navigate to the Zones tab and click on your local zone (e.g., homelab.lan).
   3. Create an A Record to establish the primary proxy destination:
   * Name: proxy
      * IP Address: 10.100.10.3
   4. Create a CNAME Record to handle the wildcard routing:
   * Name: *
      * CNAME: proxy.homelab.lan
   
Once saved, any request for anything.homelab.lan will resolve to the Nginx reverse proxy at 10.100.10.3.
------------------------------
## Step 2: Nginx Reverse Proxy Configuration
Nginx acts as the traffic controller. It listens on port 80 for incoming domain requests, reads the hostname requested by your browser, and forwards the connection to the correct backend IP and port.
Add the following server blocks to your Nginx configuration.
## 1. Technitium DNS Console Link

server {
    listen 80;
    server_name dns.homelab.lan;

    location / {
        proxy_pass http://10.100.10.2:5380;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

## 2. Vaultwarden Link

server {
    listen 80;
    server_name vault.homelab.lan;

    location / {
        proxy_pass http://10.100.10.4:80; # Replace with your actual Vaultwarden IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

## 3. Grafana Link

server {
    listen 80;
    server_name grafana.homelab.lan;

    location / {
        proxy_pass http://10.100.10.5:3000; # Replace with your actual Grafana IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

------------------------------
## Step 3: Central Dashboard (The Portal)
To prevent having to memorize or bookmark 10+ different subdomains, deploy a lightweight homepage dashboard (such as Homepage, Flame, or Dashy) on your network.

   1. Create a server block in Nginx mapping dash.homelab.lan (or just the root domain homelab.lan) to your dashboard application container.
   2. Inside the dashboard configuration file, layout your categories and link directly to your clean domain strings:

- Infrastructure:
    - Technitium DNS:
        href: http://homelab.lan
        description: Local Domain Management
    - Nginx Proxy Manager:
        href: http://homelab.lan
        description: Reverse Proxy Router

- Monitoring & Security:
    - Vaultwarden:
        href: http://homelab.lan
        description: Password Vault
    - Grafana:
        href: http://homelab.lan
        description: Metrics and Log Analytics

------------------------------
## Advantages of This Enterprise Blueprint

* Zero Container Tweak Requirements: Applications serve data from their default root paths. Web assets, scripts, and stylesheets load seamlessly without rewriting file paths.
* Security Context Isolation: Browsers isolate cookies, local storage, and session data by subdomain. Your security credentials for Vaultwarden remain isolated from other potentially vulnerable homelab containers.
* Low Maintenance Overheads: Adding an 11th application to your ecosystem requires zero modifications in Technitium DNS. You simply insert a standard five-line server definition block inside your Nginx configuration.
