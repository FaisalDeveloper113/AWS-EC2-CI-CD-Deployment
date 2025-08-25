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
