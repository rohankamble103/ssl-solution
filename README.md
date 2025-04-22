# SSL Setup with acme.sh for taskhive.co.in
This guide documents the complete process of issuing an SSL certificate using the acme.sh script for the domain taskhive.co.in, including troubleshooting the common validation errors encountered with Let's Encrypt and ZeroSSL.

## 🛠️ Problem Faced
Many certificate providers require domain ownership validation via DNS or webroot file verification. In my case, the second-stage validation consistently failed—most likely due to a delay in DNS propagation or webroot not being served correctly.

To work around this and gain full control, I decided to issue certificates using the lightweight and powerful acme.sh client.

✅ Solution Using acme.sh
**1. Install acme.sh**
```bash
   curl https://get.acme.sh | sh
```
**2. Issue Certificate via Webroot Method**
```bash
~/.acme.sh/acme.sh --issue -d taskhive.co.in --webroot /var/www/html
```
🔍 Ensure:

Your server is running and serving files from /var/www/html.

The domain points to your public IP.

.well-known/acme-challenge is accessible publicly.

**3. Installing the Certificate**
You can install the certificate using:

```bash
~/.acme.sh/acme.sh --install-cert -d taskhive.co.in \
--key-file /etc/ssl/private/taskhive.key \
--fullchain-file /etc/ssl/certs/taskhive.fullchain.pem \
--reloadcmd "systemctl reload nginx"
```
📝 You can also copy the generated files manually:

```bash
cp ~/.acme.sh/domainfoldername/*.pem /etc/ssl/certs/
cp ~/.acme.sh/domainfoldername/*.key /etc/ssl/private/
```
**4. Nginx SSL Configuration**
Update your Nginx site block like so:

```nginx
server {
    listen 80;
    server_name domainname.in www.domainname.in;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name domainnamein www.domainname.in;

    ssl_certificate /etc/ssl/certs/nameofkey.fullchain.pem;
    ssl_certificate_key /etc/ssl/private/nameofkey.key;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
```

**5. Optional: Backend API with Custom Subdomain**
For serving your Node/Express backend on port 5000:

```nginx
server {
    listen 443 ssl;
    server_name domainname.co.in;

    ssl_certificate /etc/ssl/certs/path to file.fullchain.pem;
    ssl_certificate_key /etc/ssl/private/path and name of .key;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

🔄 Automate Renewal
Add this cron job:

```bash
crontab -e
```
Then:
cron
```
0 0 * * * /root/.acme.sh/acme.sh --cron --home /root/.acme.sh
```
This ensures renewal every day at midnight.

##📸 Screenshots & Setup Walkthrough
Below are the screenshots showing:
    
![Screenshot 2025-04-22 162322](https://github.com/user-attachments/assets/039637a7-d076-4bbc-b88d-b7968ec46925)

The error I was facing with other SSL providers (Let's Encrypt/ZeroSSL).

How I resolved it using acme.sh.

Terminal output for each critical step.

![Screenshot 2025-04-22 162322](https://github.com/user-attachments/assets/2115e98e-1121-4cd2-a063-0fe83500eeac)

![Screenshot 2025-04-22 171715](https://github.com/user-attachments/assets/0bb19211-ebcc-4bfa-9ec8-49f8b68e6b92)

![Screenshot 2025-04-22 171829](https://github.com/user-attachments/assets/ca986828-cc32-41bb-8c4d-e2bbea640310)

![Screenshot 2025-04-22 161944](https://github.com/user-attachments/assets/05510db9-1a02-4773-ace5-1bc62dd72551)


# 🧹 Can I delete .acme.sh folder?
You should not delete the .acme.sh directory if you want automatic renewal. It contains:

## Renewal hooks

## Account keys

## Domain configs

Instead, copy the certs elsewhere if needed, but keep .acme.sh intact for automation.

💡 Skills Demonstrated
Troubleshooting domain validation & SSL verification failures.

Understanding of Nginx reverse proxy, port-based app routing.

Certificate lifecycle management using acme.sh.

Automation via cron jobs.

System-level file and service management (SSL directory structure, Nginx reloads).
