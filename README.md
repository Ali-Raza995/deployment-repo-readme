Deployment Guide: Next.js + PostgreSQL on EC2 with Nginx and SSL
This document outlines the steps to deploy a Next.js application connected to a PostgreSQL database on an EC2 instance. It uses Nginx as a reverse proxy and Certbot for SSL.

Prerequisites
AWS EC2 instance: Ubuntu 20.04 or later.
Domain name: Pointed to your EC2 instance's public IP.
PostgreSQL database: Running locally on the EC2 instance or hosted elsewhere (e.g., AWS RDS).
Nginx installed: Used for reverse proxying.
Node.js installed: Compatible with your Next.js app (e.g., Node.js 18+).
Certbot installed: For securing the domain with SSL.
Step 1: Setting Up PostgreSQL
Install PostgreSQL:

bash
Copy code
sudo apt update
sudo apt install postgresql postgresql-contrib
Set up a new database and user:

bash
Copy code
sudo -u postgres psql
CREATE DATABASE myappdb;
CREATE USER myappuser WITH PASSWORD 'securepassword';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO myappuser;
\q
Allow remote connections (if necessary):

Edit PostgreSQL config:

bash
Copy code
sudo nano /etc/postgresql/12/main/postgresql.conf
Find and set:

plaintext
Copy code
listen_addresses = '*'
Update pg_hba.conf:

bash
Copy code
sudo nano /etc/postgresql/12/main/pg_hba.conf
Add:

plaintext
Copy code
host    all             all             0.0.0.0/0               md5
Restart PostgreSQL:

bash
Copy code
sudo systemctl restart postgresql
Step 2: Deploy Next.js on EC2
Clone your repository:

bash
Copy code
git clone https://github.com/yourusername/yourrepo.git
cd yourrepo
Install dependencies:

bash
Copy code
npm install
Set up environment variables:

Create a .env file:
plaintext
Copy code
DATABASE_URL=postgresql://myappuser:securepassword@localhost:5432/myappdb
NEXT_PUBLIC_API_URL=https://lachlan-app.impleko.com
Replace DATABASE_URL and NEXT_PUBLIC_API_URL with your actual database connection string and domain.
Build your Next.js app:

bash
Copy code
npm run build
Run the app in production: Install pm2 to keep your app running:

bash
Copy code
sudo npm install -g pm2
pm2 start npm --name "nextjs-app" -- start
pm2 save
Step 3: Configure Nginx for Reverse Proxy
Install Nginx:

bash
Copy code
sudo apt install nginx
Set up a server block for your domain:

bash
Copy code
sudo nano /etc/nginx/sites-available/lachlan-app
Add the following configuration:

nginx
Copy code
server {
    server_name my-domain;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 80;
}
Enable the configuration:

bash
Copy code
sudo ln -s /etc/nginx/sites-available/lachlan-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
Step 4: Enable HTTPS with Certbot
Install Certbot:

bash
Copy code
sudo apt install certbot python3-certbot-nginx
Obtain an SSL certificate:

bash
Copy code
sudo certbot --nginx -d lachlan-app.impleko.com
Auto-renew SSL: Certbot auto-renewal is enabled by default. You can test it with:

bash
Copy code
sudo certbot renew --dry-run
Step 5: Verify the Deployment
Visit https://lachlan-app.impleko.com to ensure your app is accessible over HTTPS.
Check the PostgreSQL connection and ensure data is being saved/retrieved correctly.
Step 6: Troubleshooting & Maintenance
Logs:

Check Nginx logs:
bash
Copy code
sudo tail -f /var/log/nginx/error.log
Check PM2 logs:
bash
Copy code
pm2 logs
Restart services:

Restart Nginx:
bash
Copy code
sudo systemctl restart nginx
Restart the app:
bash
Copy code
pm2 restart nextjs-app
Notes
Always push updates to your repository and pull them on the EC2 instance.
After pulling changes, rebuild the app:
bash
Copy code
npm run build
pm2 restart nextjs-app
This README should be placed in your GitHub repository for future reference.
