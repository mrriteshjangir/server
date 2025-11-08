# 🚀 Ubuntu 22.04 Fullstack Server Setup Guide (Node.js, Next.js, React, MongoDB)

This single guide helps you configure a **secure, high-performance, production-ready Ubuntu 22.04 server** for hosting Node.js, Next.js, or React apps — with Nginx, PM2, SSL, MongoDB, and basic security hardening.

> ⚙️ Replace placeholders like `your_domain.com`, `api.your_domain.com`, and ports as needed.

---

## 1. Update & Essentials

sudo apt update && sudo apt upgrade -y  
sudo timedatectl set-timezone Asia/Kolkata  
sudo apt install -y curl git ufw build-essential software-properties-common

---

## 2. Install & Enable Nginx

sudo apt install -y nginx  
sudo systemctl enable nginx  
sudo systemctl start nginx  
systemctl status nginx

---

## 3. Configure Firewall (UFW)

sudo ufw allow OpenSSH  
sudo ufw allow 'Nginx HTTP'  
sudo ufw enable  
sudo ufw status

---

## 4. Create Web Directory

sudo mkdir -p /var/www/your_domain  
sudo chown -R $USER:$USER /var/www/your_domain  
sudo chmod -R 755 /var/www/your_domain

---

## 5. Nginx Configuration

sudo nano /etc/nginx/sites-available/your_domain

Paste the following config:

server {
    listen 80;
    listen [::]:80;
    add_header Server "MyCompany Server";
    server_name api.your_domain.com;
    client_max_body_size 1024M;

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

## 6. SSL with Let's Encrypt (Certbot)

sudo apt install -y certbot python3-certbot-nginx  
sudo certbot --nginx -d your_domain.com -d www.your_domain.com  
sudo certbot renew --dry-run

---

## 7. Install Node.js (via NVM)

sudo apt install curl -y  
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash  
source ~/.bashrc  
nvm install 18  
nvm use 18  
node -v  
npm -v

---

## 8. PM2 Process Manager

npm install -g pm2  
pm2 start index.js --name "api-app" -i max --node-args="--max-old-space-size=512"  
pm2 save  
pm2 startup  

*(Follow the command output to enable PM2 startup service.)*

---

## 9. PM2 Commands

pm2 list  
pm2 logs  
pm2 restart all  
pm2 delete all  
pm2 monit  

---

## 10. Install MongoDB (Latest Version)

Import MongoDB public key & add repository:

curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor  
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

Install MongoDB:

sudo apt update  
sudo apt install -y mongodb-org  

Start and enable service:

sudo systemctl start mongod  
sudo systemctl enable mongod  
sudo systemctl status mongod  

Check version:

mongod --version

---

## 11. MongoDB Security (Remote Access + Auth)

Edit MongoDB config:

sudo nano /etc/mongod.conf

Find `bindIp` and modify:
bindIp: 0.0.0.0

Enable authentication:
security:
  authorization: enabled

Save and restart:
sudo systemctl restart mongod

Create admin user:
mongosh
use admin
db.createUser({ user: "adminUser", pwd: "StrongPassword123", roles: [ { role: "root", db: "admin" } ] })
exit

Allow MongoDB port in firewall (optional, restrict by IP if needed):
sudo ufw allow 27017/tcp

---

## 12. Security & Optimization

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

Optional Swap (for low-memory VPS):
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

---

## 13. Folder Structure

/var/www/
 ├── api.your_domain.com/
 │   ├── index.js
 │   ├── package.json
 │   └── ...
 ├── your_domain.com/
 │   ├── .next/
 │   ├── build/
 │   └── ...

---

## 14. Maintenance Commands

sudo nginx -t  
sudo systemctl reload nginx  
pm2 restart all  
sudo certbot renew  
sudo ufw status numbered  
sudo journalctl -xe  

---

## 15. Monitoring (Optional)

### Netdata (real-time metrics)
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
Access via: http://your_server_ip:19999

### PM2 Monitoring Dashboard
pm2 monit

---

## 16. Backup Setup (Optional)

Install Rclone:
sudo apt install rclone -y

Backup web + pm2 data:
sudo tar -czvf /root/server-backup-$(date +%F).tar.gz /var/www ~/.pm2

Upload to cloud:
rclone copy /root/server-backup-*.tar.gz remote:server-backups

---

## 17. MongoDB Backup (Optional)

Create DB backup:
mongodump --out /root/mongo-backups/$(date +%F)

Restore backup:
mongorestore /root/mongo-backups/<date>

(Optional) Compress backup:
tar -czvf /root/mongo-backup-$(date +%F).tar.gz /root/mongo-backups/$(date +%F)

---

## 18. Final Notes

✅ Ensure DNS records point to your server before SSL setup.  
✅ Run `pm2 save` after app changes.  
✅ Use strong MongoDB credentials & firewall rules.  
✅ Reboot after setup:  
sudo reboot