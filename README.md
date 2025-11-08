# 🚀 Ubuntu 22.04 Fullstack Server Setup Guide
**(Node.js | Next.js | React | Nginx | PM2 | Certbot | GitHub Deploy)**

This guide provides a complete step-by-step setup for a **secure, reliable, and production-ready Ubuntu 22.04 server**.  
It assumes you will use:
- `/mnt/server` → for backend (Node.js / API)
- `/mnt/web` → for frontend (Next.js / React)

---

## 🧱 1. System Setup & Essentials

```bash
sudo apt update && sudo apt upgrade -y
```

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

```bash
sudo apt install -y curl git ufw build-essential software-properties-common ca-certificates gnupg lsb-release unzip
```

---

## 🌐 2. Nginx Installation & Configuration

```bash
sudo apt install -y nginx
```

```bash
sudo systemctl enable nginx
```

```bash
sudo systemctl start nginx
```

```bash
sudo systemctl status nginx
```

---

## 🔥 3. Firewall (UFW)

```bash
sudo ufw allow OpenSSH
```

```bash
sudo ufw allow 'Nginx Full'
```

```bash
sudo ufw --force enable
```

```bash
sudo ufw status
```

---

## 📁 4. Create Folder Structure

```bash
mkdir -p /mnt/server
```

```bash
mkdir -p /mnt/web
```

```bash
sudo chown -R $USER:$USER /mnt
```

```bash
sudo chmod -R 755 /mnt
```

---

## 💾 5. Swap Memory Setup

Create swap file (recommended: 2GB for servers with 1-2GB RAM, 4GB for servers with 4GB+ RAM):

```bash
sudo fallocate -l 2G /swapfile
```

**Note:** Adjust size as needed (e.g., `-l 4G` for 4GB swap)

Set proper permissions:

```bash
sudo chmod 600 /swapfile
```

Format as swap:

```bash
sudo mkswap /swapfile
```

Enable swap:

```bash
sudo swapon /swapfile
```

Make swap permanent (survives reboot):

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Verify swap is active:

```bash
free -h
```

Check swap usage:

```bash
swapon --show
```

**Optional:** Optimize swap usage (reduce swappiness - default is 60):

```bash
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

```bash
sudo sysctl vm.swappiness=10
```

---

## ⚙️ 6. Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/your_domain
```

Paste this configuration:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name api.your_domain.com;
    client_max_body_size 1024M;
    add_header Server "MyCompany Server";

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
    }
}

server {
    listen 80;
    listen [::]:80;

    server_name your_domain.com www.your_domain.com;
    client_max_body_size 1024M;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
    }

    location /static/ {
        expires 1d;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
}
```

Enable and test:

```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```

---

## 🔒 7. SSL with Certbot (HTTPS Setup)

Install Certbot and Nginx plugin:

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Obtain certificates for main domain:

```bash
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```

Obtain certificate for API domain (optional):

```bash
sudo certbot --nginx -d api.your_domain.com
```

Test auto-renewal:

```bash
sudo certbot renew --dry-run
```

Renew manually (monthly check):

```bash
sudo certbot renew && sudo systemctl reload nginx
```

---

## 🧠 8. Node.js Installation (via NVM)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

```bash
source ~/.bashrc
```

```bash
nvm install 18
```

```bash
nvm use 18
```

```bash
nvm alias default 18
```

```bash
node -v
```

```bash
npm -v
```

---

## ⚡ 9. PM2 Process Manager

```bash
npm install -g pm2
```

```bash
pm2 start /mnt/server/index.js --name "api-server" -i max --node-args="--max-old-space-size=512"
```

```bash
pm2 save
```

```bash
pm2 startup
```

**Note:** Run the command printed by `pm2 startup` to enable PM2 on system boot.

---

## 🧰 10. PM2 Commands

```bash
pm2 list
```

```bash
pm2 logs
```

```bash
pm2 restart all
```

```bash
pm2 delete all
```

```bash
pm2 monit
```

```bash
pm2 stop all
```

```bash
pm2 reload all
```

---

## 💻 11. Import Project from GitHub

### Option 1: Using HTTPS with Personal Access Token

```bash
cd /mnt/server
```

```bash
git clone https://username:github_token@github.com/username/projectname.git
```

```bash
cd /mnt/web
```

```bash
git clone https://username:github_token@github.com/username/frontend.git
```

**Note:** Replace `username` and `github_token` with your actual GitHub credentials. Use a Personal Access Token instead of password.

### Option 2: Using SSH Key (Recommended)

Generate SSH key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Copy public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add this key to GitHub → **Settings → SSH and GPG Keys → New SSH Key**

Clone repositories:

```bash
cd /mnt/server
```

```bash
git clone git@github.com:username/projectname.git
```

```bash
cd /mnt/web
```

```bash
git clone git@github.com:username/frontend.git
```

---

## 🔐 12. Security Hardening

### Disable Nginx Version Info

```bash
sudo nano /etc/nginx/nginx.conf
```

Inside `http {}` block, add:
```nginx
server_tokens off;
server_names_hash_bucket_size 64;
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

### Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

```bash
sudo systemctl enable fail2ban
```

```bash
sudo systemctl start fail2ban
```

### Enable Automatic Security Updates

```bash
sudo apt install unattended-upgrades -y
```

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

## 📂 13. Folder Structure

```
/mnt/
 ├── server/        → Node.js backend
 ├── web/           → Next.js / React frontend
 └── logs/          → App & system logs
```

Create additional directories:

```bash
mkdir -p /mnt/logs
```

---

## 🧼 14. Maintenance Commands

Test Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

Restart PM2 processes:

```bash
pm2 restart all
```

Renew SSL certificates:

```bash
sudo certbot renew
```

Check firewall status:

```bash
sudo ufw status numbered
```

View system logs:

```bash
sudo journalctl -xe
```

Check disk usage:

```bash
df -h
```

---

## 📊 15. Monitoring (Optional)

### Netdata (Real-time System Monitor)

```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

Access at: `http://your_server_ip:19999`

### PM2 Dashboard

```bash
pm2 monit
```

### System Resources

```bash
htop
```

Install htop:

```bash
sudo apt install htop -y
```

---

## 💾 16. Backup Setup (Optional)

### Install Rclone

```bash
sudo apt install rclone -y
```

### Backup Web & Server Files

```bash
sudo tar -czvf /root/server-backup-$(date +%F).tar.gz /mnt/server /mnt/web ~/.pm2
```

### Upload to Cloud Storage

```bash
rclone copy /root/server-backup-*.tar.gz remote:server-backups
```

---

## ✅ 17. Final Notes

- ✅ Ensure DNS records point to your server IP before running Certbot
- ✅ Always run `pm2 save` after changes to persist process list
- ✅ Use HTTPS-only access once SSL is active
- ✅ Use GitHub Personal Access Tokens instead of plain passwords
- ✅ Keep system updated regularly: `sudo apt update && sudo apt upgrade -y`
- ✅ Monitor logs regularly: `pm2 logs` and `sudo journalctl -xe`
- ✅ Test Nginx configuration before reloading: `sudo nginx -t`
- ✅ Reboot once after setup for clean start:

```bash
sudo reboot
```

---

## 🔗 Quick Reference

### Essential Commands

| Task | Command |
|------|---------|
| Check Nginx status | `sudo systemctl status nginx` |
| Check PM2 status | `pm2 list` |
| Check swap usage | `free -h` or `swapon --show` |
| View PM2 logs | `pm2 logs` |
| Restart all services | `pm2 restart all && sudo systemctl restart nginx` |
| Check firewall | `sudo ufw status` |
| System update | `sudo apt update && sudo apt upgrade -y` |

---

**Last Updated:** 2024  
**Tested on:** Ubuntu 22.04 LTS
