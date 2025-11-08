# 🚀 Ubuntu 22.04 Fullstack Server Setup Guide
**(Node.js | Next.js | React | MongoDB | Nginx | PM2 | Certbot | GitHub Deploy)**

This guide provides a complete step-by-step setup for a **secure, reliable, and production-ready Ubuntu 22.04 server**.  
It assumes you will use:
- `/mnt/server` → for backend (Node.js / API)
- `/mnt/web` → for frontend (Next.js / React)

---

## 🧱 1. System Setup & Essentials

sudo apt update && sudo apt upgrade -y  
sudo timedatectl set-timezone Asia/Kolkata  
sudo apt install -y curl git ufw build-essential software-properties-common ca-certificates gnupg lsb-release unzip

---

## 🌐 2. Nginx Installation & Configuration

sudo apt install -y nginx  
sudo systemctl enable nginx  
sudo systemctl start nginx  
systemctl status nginx  

---

## 🔥 3. Firewall (UFW)

sudo ufw allow OpenSSH  
sudo ufw allow 'Nginx Full'  
sudo ufw --force enable  
sudo ufw status

---

## 📁 4. Create Folder Structure

mkdir -p /mnt/server  
mkdir -p /mnt/web  
sudo chown -R $USER:$USER /mnt  
sudo chmod -R 755 /mnt  

---

## ⚙️ 5. Nginx Configuration

sudo nano /etc/nginx/sites-available/your_domain

Paste this config:

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

Enable and test:
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/  
sudo nginx -t  
sudo systemctl restart nginx  

---

## 🔒 6. SSL with Certbot (HTTPS Setup)

Install Certbot and Nginx plugin:

sudo apt install -y certbot python3-certbot-nginx

Obtain certificates:
sudo certbot --nginx -d your_domain.com -d www.your_domain.com

(Optional API domain):
sudo certbot --nginx -d api.your_domain.com

Test auto-renewal:
sudo certbot renew --dry-run

Renew manually (monthly check):
sudo certbot renew && sudo systemctl reload nginx

---

## 🧠 7. Node.js Installation (via NVM)

sudo apt install curl -y  
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash  
source ~/.bashrc  
nvm install 18  
nvm use 18  
node -v  
npm -v

---

## ⚡ 8. PM2 Process Manager

npm install -g pm2  
pm2 start /mnt/server/index.js --name "api-server" -i max --node-args="--max-old-space-size=512"  
pm2 save  
pm2 startup  
# (Run the command printed by pm2 startup)

---

## 🧰 9. PM2 Commands

pm2 list  
pm2 logs  
pm2 restart all  
pm2 delete all  
pm2 monit  

---

## 🗄️ 10. MongoDB Installation & Configuration

Add MongoDB repository:
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor  
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list  

Install MongoDB:
sudo apt update  
sudo apt install -y mongodb-org  

Enable and check:
sudo systemctl enable mongod  
sudo systemctl start mongod  
sudo systemctl status mongod  
mongod --version

---

## 🔐 11. Secure MongoDB (Remote Access & Auth)

sudo nano /etc/mongod.conf  

Change:
net:
  bindIp: 0.0.0.0

Add:
security:
  authorization: enabled

Restart:
sudo systemctl restart mongod  

Create admin user:
mongosh  
use admin  
db.createUser({ user: "adminUser", pwd: "StrongPassword123", roles: [ { role: "root", db: "admin" } ] })  
exit  

Allow MongoDB port:
sudo ufw allow 27017/tcp  

---

## 💻 12. Import Project from GitHub

### Option 1: Using HTTPS with username/token
cd /mnt/server  
git clone https://username:github_token@github.com/username/projectname.git  

cd /mnt/web  
git clone https://username:github_token@github.com/username/frontend.git  

*(Replace username and token with your actual GitHub credentials.)*

---

### Option 2: Using SSH Key (Recommended)
Generate SSH key:
ssh-keygen -t ed25519 -C "your_email@example.com"

Copy key:
cat ~/.ssh/id_ed25519.pub

Add this key to GitHub → **Settings → SSH and GPG Keys → New SSH Key**

Then clone:
cd /mnt/server  
git clone git@github.com:username/projectname.git  
cd /mnt/web  
git clone git@github.com:username/frontend.git

---

## 🔐 13. Security Hardening

Disable Nginx version info:
sudo nano /etc/nginx/nginx.conf  
Inside `http {}` add:
server_tokens off;
server_names_hash_bucket_size 64;

Install Fail2Ban:
sudo apt install fail2ban -y

Enable automatic security updates:
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

Add Swap (for low-memory VPS):
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

---

## 📂 14. Folder Structure

/mnt/
 ├── server/        → Node.js backend  
 ├── web/           → Next.js / React frontend  
 ├── mongo-backups/ → MongoDB backups  
 └── logs/          → App & system logs  

---

## 🧼 15. Maintenance Commands

sudo nginx -t  
sudo systemctl reload nginx  
pm2 restart all  
sudo certbot renew  
sudo ufw status numbered  
sudo journalctl -xe  

---

## 📊 16. Monitoring (Optional)

### Netdata (Real-time System Monitor)
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
→ Access: http://your_server_ip:19999  

### PM2 Dashboard
pm2 monit  

---

## 💾 17. Backup Setup (Optional)

Install Rclone:
sudo apt install rclone -y

Backup web & server:
sudo tar -czvf /root/server-backup-$(date +%F).tar.gz /mnt/server /mnt/web ~/.pm2  

Upload to cloud:
rclone copy /root/server-backup-*.tar.gz remote:server-backups  

---

## 🧮 18. MongoDB Backup (Manual or Cron)

Manual:
mongodump --out /mnt/mongo-backups/$(date +%F)

Restore:
mongorestore /mnt/mongo-backups/<date>

Compress:
tar -czvf /root/mongo-backup-$(date +%F).tar.gz /mnt/mongo-backups/$(date +%F)

To automate daily at 2 AM:
sudo crontab -e
0 2 * * * mongodump --out /mnt/mongo-backups/$(date +\%F)

---

## ✅ 19. Final Notes

- Ensure DNS records point to your server IP before running Certbot.  
- Always run `pm2 save` after changes.  
- Use HTTPS-only access once SSL is active.  
- Use GitHub tokens instead of plain passwords for automation.  
- Reboot once after setup for clean start:

sudo reboot

---