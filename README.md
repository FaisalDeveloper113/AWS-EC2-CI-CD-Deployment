# AWS-EC2-CI-CD-Deployment
access the server using ssh command
ssh -i "QuantFarming-backend.pem" ubuntu@ec2-51-20-93-52.eu-north-1.compute.amazonaws.com

* Create a ubuntu server on aws
* setup github action (run all commands on server and enable the runner)

run these commands to run your nodejs app

# Go and check work folder
# Install Nodejs
sudo apt update
sudo apt-get install -y nodejs npm
sudo apt-get install -y nginx
sudo npm i -g pm2
cd /etc/nginx/sites-available
sudo nano default

location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
}

sudo systemctl restart nginx
cd /path/to/your/app
pm2 start server.js --name=apiserver
pm2 restart apiserver
