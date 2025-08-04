# 🌐 Self-Hosting SSL-Enabled n8n on a Linux Server

This guide walks you through the process of self-hosting [n8n](https://n8n.io) — a free and open-source workflow automation tool — on a Linux server using **Docker**, **Nginx**, and **Certbot** for SSL with a **custom domain or subdomain**.

> ✅ Perfect for developers, hobbyists, and teams looking to run n8n securely on their own infrastructure.

---

## 🐳 Step 1: Install Docker

1. **Update your package index:**

   ```bash
   sudo apt update
   ```

2. **Install Docker:**

   ```bash
   sudo apt install docker.io
   ```

3. **Start the Docker service:**

   ```bash
   sudo systemctl start docker
   ```

4. **Enable Docker to start at boot:**

   ```bash
   sudo systemctl enable docker
   ```

---

## 🚀 Step 2: Run n8n in Docker

Run the following command to start n8n. Replace `your-domain.com` with your actual domain or subdomain.

### 🔗 For a domain:

```bash
sudo docker run -d --restart unless-stopped -it \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST="your-domain.com" \
  -e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
  -e WEBHOOK_URL="https://your-domain.com/" \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```

### 🌐 For a subdomain:

```bash
sudo docker run -d --restart unless-stopped -it \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST="subdomain.your-domain.com" \
  -e WEBHOOK_TUNNEL_URL="https://subdomain.your-domain.com/" \
  -e WEBHOOK_URL="https://subdomain.your-domain.com/" \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```

> 🔒 This will run n8n in a Docker container and expose it on port `5678`, with persistent storage and HTTPS-ready webhook URLs.

---

## 🌐 Step 3: Install Nginx

We'll use Nginx as a reverse proxy in front of n8n.

1. **Install Nginx:**

   ```bash
   sudo apt install nginx
   ```

---

## ⚙️ Step 4: Configure Nginx

1. **Create a new Nginx config file:**

   ```bash
   sudo nano /etc/nginx/sites-available/n8n.conf
   ```

2. **Add the following configuration:**

   ```nginx
   server {
       listen 80;
       server_name your-domain.com;  # Use subdomain.your-domain.com if applicable

       location / {
           proxy_pass http://localhost:5678;
           proxy_http_version 1.1;
           chunked_transfer_encoding off;
           proxy_buffering off;
           proxy_cache off;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto https;
           proxy_read_timeout 86400;
       }
   }
   ```

   Replace `your-domain.com` with your actual domain or subdomain.

3. **Enable the configuration and restart Nginx:**

   ```bash
   sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

---

## 🔐 Step 5: Secure with SSL using Certbot

We'll use Certbot to obtain a free SSL certificate from Let’s Encrypt.

1. **Install Certbot and the Nginx plugin:**

   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. **Run Certbot to obtain the SSL certificate:**

   ```bash
   sudo certbot --nginx -d your-domain.com
   ```

   Replace with your subdomain if needed, e.g., `subdomain.your-domain.com`.

3. **Follow the prompts** to complete the SSL setup.

> ✅ Once completed, your n8n instance will be accessible at **[https://your-domain.com](https://your-domain.com)** with a valid SSL certificate.

---

### 🔁 What happens to your Nginx config?

Certbot will automatically update your `/etc/nginx/sites-available/n8n.conf` file to include the SSL blocks. Your updated config will resemble the following:

![SSL-enabled Nginx config](/ssl.png)
*(Replace with actual screenshot if available)*

---

## 🧠 Final Thoughts

By following this guide, you’ve successfully:

* Hosted **n8n** on your own Linux server
* Secured it with **HTTPS**
* Set up a robust **Docker + Nginx + Certbot** architecture

> 💡 If this guide helped you, consider giving the [n8n GitHub repo](https://github.com/n8n-io/n8n) a ⭐ and sharing this guide to help others in the community!

---

Let me know if you'd like a Markdown file version or if you're planning to publish it on GitHub/Blog — I can tailor it accordingly.
