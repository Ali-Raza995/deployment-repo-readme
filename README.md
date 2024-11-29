<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
  <h1>Deployment Guide: Next.js + PostgreSQL on EC2 with Nginx and SSL</h1>
  <p>
    This document provides step-by-step instructions to deploy a Next.js application with PostgreSQL on an EC2 instance using Nginx as a reverse proxy and Certbot for SSL.
  </p>
  <hr />

  <h2>Prerequisites</h2>
  <ul>
    <li><strong>AWS EC2 instance:</strong> Running Ubuntu 20.04 or later.</li>
    <li><strong>Domain name:</strong> Pointed to your EC2 instance's public IP.</li>
    <li><strong>PostgreSQL database:</strong> Either local or hosted (e.g., AWS RDS).</li>
    <li><strong>Nginx:</strong> Installed for reverse proxy.</li>
    <li><strong>Node.js:</strong> Installed, compatible with your Next.js application (e.g., Node.js 18+).</li>
    <li><strong>Certbot:</strong> Installed for securing your domain with SSL.</li>
  </ul>
  <hr />

  <h2>Step 1: Setting Up PostgreSQL</h2>
  <h3>Install PostgreSQL</h3>
  <pre>
<code>
sudo apt update
sudo apt install postgresql postgresql-contrib
</code>
  </pre>
  
  <h3>Create Database and User</h3>
  <pre>
<code>
sudo -u postgres psql
CREATE DATABASE myappdb;
CREATE USER myappuser WITH PASSWORD 'securepassword';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO myappuser;
\q
</code>
  </pre>

  <h3>Allow Remote Connections (Optional)</h3>
  <ol>
    <li>Edit PostgreSQL configuration:
      <pre>
<code>
sudo nano /etc/postgresql/12/main/postgresql.conf
</code>
      </pre>
      Update the following line:
      <pre>
<code>
listen_addresses = '*'
</code>
      </pre>
    </li>
    <li>Modify <code>pg_hba.conf</code>:
      <pre>
<code>
sudo nano /etc/postgresql/12/main/pg_hba.conf
</code>
      </pre>
      Add this line:
      <pre>
<code>
host    all             all             0.0.0.0/0               md5
</code>
      </pre>
    </li>
    <li>Restart PostgreSQL:
      <pre>
<code>
sudo systemctl restart postgresql
</code>
      </pre>
    </li>
  </ol>
  <hr />

  <h2>Step 2: Deploy Next.js on EC2</h2>
  <h3>Clone Your Repository</h3>
  <pre>
<code>
git clone https://github.com/yourusername/yourrepo.git
cd yourrepo
</code>
  </pre>
  
  <h3>Install Dependencies</h3>
  <pre>
<code>
npm install
</code>
  </pre>
  
  <h3>Set Up Environment Variables</h3>
  <p>Create a <code>.env</code> file and add the following:</p>
  <pre>
<code>
DATABASE_URL=postgresql://myappuser:securepassword@localhost:5432/myappdb
NEXT_PUBLIC_API_URL=https://lachlan-app.impleko.com
</code>
  </pre>

  <h3>Build and Start the App</h3>
  <pre>
<code>
npm run build
sudo npm install -g pm2
pm2 start npm --name "nextjs-app" -- start
pm2 save
</code>
  </pre>
  <hr />

  <h2>Step 3: Configure Nginx as Reverse Proxy</h2>
  <h3>Install Nginx</h3>
  <pre>
<code>
sudo apt install nginx
</code>
  </pre>

  <h3>Create a Server Block</h3>
  <pre>
<code>
sudo nano /etc/nginx/sites-available/lachlan-app
</code>
  </pre>
  <p>Add the following configuration:</p>
  <pre>
<code>
server {
    server_name lachlan-app.impleko.com;

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
</code>
  </pre>
  
  <h3>Enable Configuration</h3>
  <pre>
<code>
sudo ln -s /etc/nginx/sites-available/lachlan-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
</code>
  </pre>
  <hr />

  <h2>Step 4: Enable HTTPS with Certbot</h2>
  <h3>Install Certbot</h3>
  <pre>
<code>
sudo apt install certbot python3-certbot-nginx
</code>
  </pre>
  
  <h3>Obtain an SSL Certificate</h3>
  <pre>
<code>
sudo certbot --nginx -d lachlan-app.impleko.com
</code>
  </pre>
  
  <h3>Test Auto-Renewal</h3>
  <pre>
<code>
sudo certbot renew --dry-run
</code>
  </pre>
  <hr />

  <h2>Step 5: Verify the Deployment</h2>
  <ol>
    <li>Visit <code>https://lachlan-app.impleko.com</code> to ensure your application is live.</li>
    <li>Confirm that your PostgreSQL database is connected and operational.</li>
  </ol>
  <hr />

  <h2>Step 6: Troubleshooting & Maintenance</h2>
  <h3>Logs</h3>
  <ul>
    <li><strong>Nginx Logs:</strong>
      <pre>
<code>
sudo tail -f /var/log/nginx/error.log
</code>
      </pre>
    </li>
    <li><strong>PM2 Logs:</strong>
      <pre>
<code>
pm2 logs
</code>
      </pre>
    </li>
  </ul>
  
  <h3>Restart Services</h3>
  <ul>
    <li>Restart Nginx:
      <pre>
<code>
sudo systemctl restart nginx
</code>
      </pre>
    </li>
    <li>Restart the App:
      <pre>
<code>
pm2 restart nextjs-app
</code>
      </pre>
    </li>
  </ul>
  <hr />

  <h2>Notes</h2>
  <ul>
    <li>Push updates to your repository and pull them on the EC2 instance.</li>
    <li>After pulling changes, rebuild the app:
      <pre>
<code>
npm run build
pm2 restart nextjs-app
</code>
      </pre>
    </li>
  </ul>

  <p>This guide ensures a smooth deployment process for your Next.js application on EC2. Save it in your GitHub repository for future use!</p>
</body>
</html>
