---

# AWS-EC2-CI-CD-Deployment

A concise guide to deploy a Node.js app to an AWS EC2 instance and run it behind Nginx with PM2, using GitHub Actions.

Note: The SSH command shown is an example to connect to your server. Adapt the host, user, and key to your setup.

---

## 1) SSH Example: Connect to Your EC2 Instance

Example command to connect (replace with your actual values):

```bash
ssh -i "QuantFarming-backend.pem" ubuntu@ec2-51-20-93-52.eu-north-1.compute.amazonaws.com
```

- This is an example connection command. Use your own PEM key, username, and host.
- Ensure security groups allow SSH (port 22) from your IP.

---

## 2) Provisioning the EC2 Instance (Ubuntu)

- Create an Ubuntu server on AWS (per your setup).
- After provisioning, connect via SSH (as shown above) and proceed with server-side setup as needed.

Optional best practices (no extra commands added here):
- Create a non-root user and use SSH keys.
- Harden SSH (disable password auth, disable root login).

---

## 3) Go to Work Folder and Install Node.js / Nginx / PM2

These steps are meant to be run on the server. Use them exactly as you provided:

- Go to your application folder:
  - cd /path/to/your/app

- Install Node.js, Nginx, and PM2:

```bash
sudo apt update

sudo apt-get install -y nodejs npm

sudo apt-get install -y nginx

sudo npm i -g pm2
```

- Configure Nginx (example setup in /etc/nginx/sites-available). Edit the default server block to proxy to your app (adjust as needed):

```bash
cd /etc/nginx/sites-available
sudo nano default
```

Sample proxy block to include (as you indicated):

```
location / {
    proxy_pass http://localhost:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

- Restart Nginx:

```bash
sudo systemctl restart nginx
```

- Start and manage your Node.js app with PM2:

```bash
cd /path/to/your/app

pm2 start server.js --name=apiserver

pm2 restart apiserver
```

---

## 4) GitHub Actions

Your plan mentions using GitHub Actions to run commands on the server. Since you asked to keep only the provided commands, this README doesn’t include new Actions workflow content. When you’re ready to wire GitHub Actions, you can use either:
- A self-hosted runner on the EC2 instance (recommended for running these server-side commands), or
- A workflow that SSHs into the server to execute the above commands (via secrets for SSH ключ).

Guidance you can follow later (without adding new commands here):
- Set up a self-hosted runner on the EC2 instance and create a workflow that runs the necessary steps on that runner.
- Or configure a workflow with an SSH action to execute the provided server commands in sequence.

---

If you’d like, I can tailor a GitHub Actions workflow file to your exact repository structure using only the above commands, either with a self-hosted runner or an SSH-based approach, while keeping the content strictly to your commands.








# Backend API Deployment Guide

Complete setup guide for deploying a Python backend API on AWS EC2 with SSL certificate.

## 🏗️ Architecture

- **Backend**: Python API (FastAPI/Django) running on EC2
- **Process Manager**: PM2 for application management
- **Web Server**: Nginx as reverse proxy
- **SSL**: Let's Encrypt certificate with auto-renewal
- **Domain**: Custom subdomain with DNS configuration

## 🚀 Deployment Steps

### 1. DNS Configuration

Set up A record for your domain:
- **Type**: A Record
- **Name**: backend
- **Value**: Your EC2 public IP
- **TTL**: 1/2 Hour

### 2. Application Setup

Deploy your Python application and start with PM2:

```bash
# Install PM2 globally
npm install -g pm2

# Start your application
pm2 start app.py --name apiserver

# Check status
pm2 status
```

### 3. Nginx Configuration

Install and configure Nginx as reverse proxy:

```bash
# Install Nginx
sudo apt update
sudo apt install nginx -y

# Create site configuration
sudo nano /etc/nginx/sites-available/backend
```

Add configuration:
```nginx
server {
    listen 80;
    server_name backend.yourdomain.com;
    
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4. SSL Certificate

Install Certbot and obtain SSL certificate:

```bash
# Install Certbot
sudo apt install snapd -y
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Obtain SSL certificate
sudo certbot --nginx -d backend.yourdomain.com
```

## 🔧 Configuration Details

### PM2 Process Management

```bash
# View application logs
pm2 logs apiserver

# Monitor resources
pm2 monit

# Restart application
pm2 restart apiserver

# Save PM2 configuration
pm2 save
pm2 startup
```

### SSL Certificate Management

- **Certificate Location**: `/etc/letsencrypt/live/backend.yourdomain.com/`
- **Auto-renewal**: Enabled by default
- **Expiry**: 90 days from issuance

Test renewal:
```bash
sudo certbot renew --dry-run
```

## 🛡️ Security Configuration

### EC2 Security Group Rules

| Type  | Port | Protocol | Source    | Purpose        |
|-------|------|----------|-----------|----------------|
| SSH   | 22   | TCP      | Your IP   | Server access  |
| HTTP  | 80   | TCP      | 0.0.0.0/0 | Web traffic    |
| HTTPS | 443  | TCP      | 0.0.0.0/0 | Secure traffic |

### Nginx Security Headers

The Nginx configuration includes:
- Proper proxy headers
- Connection upgrade support
- Real IP forwarding

## 📋 Verification

### Test Your Deployment

```bash
# Test HTTP (should redirect to HTTPS)
curl -I http://backend.yourdomain.com

# Test HTTPS
curl https://backend.yourdomain.com

# Check SSL certificate
openssl s_client -connect backend.yourdomain.com:443
```

### Health Checks

```bash
# Check PM2 status
pm2 status

# Check Nginx status
sudo systemctl status nginx

# View application logs
pm2 logs apiserver --lines 50
```

## 🔄 Maintenance

### Regular Tasks

- **SSL Renewal**: Automatic (every 60 days)
- **Application Updates**: Use PM2 reload for zero-downtime
- **Log Rotation**: PM2 handles log rotation automatically

### Backup Important Files

- Application code
- Nginx configuration: `/etc/nginx/sites-available/backend`
- PM2 configuration: `pm2 save`

## 📞 Troubleshooting

### Common Issues

1. **502 Bad Gateway**: Check if PM2 app is running on correct port
2. **SSL Certificate Error**: Verify DNS propagation
3. **Connection Refused**: Check EC2 security group rules

### Useful Commands

```bash
# Check port usage
sudo netstat -tlnp | grep :5000

# Nginx configuration test
sudo nginx -t

# View SSL certificate details
sudo certbot certificates
```

## ✅ Final Result

Your API is now accessible at:
**https://backend.yourdomain.com**

- ✅ SSL-secured connection
- ✅ Automatic certificate renewal
- ✅ Production-ready configuration
- ✅ Process management with PM2
- ✅ Reverse proxy with Nginx
