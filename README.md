# Host FastAPI to AWS EC2

This is a simple example FastAPI application that pretends to be a bookstore.

# Deployment

Log into your AWS account and create an EC2 instance (`t2.micro`), using the latest stable
Ubuntu Linux AMI.

[SSH into the instance](https://aws.amazon.com/blogs/compute/new-using-amazon-ec2-instance-connect-for-ssh-access-to-your-ec2-instances/) and run these commands to update the software repository and install
our dependencies.

```bash
sudo apt-get update
sudo apt install -y python3-pip nginx
```

Clone the FastAPI server app (or create your `main.py` in Python).

```bash
git clone https://github.com/{your-repo}.git
```

Add the FastAPI configuration to NGINX's folder. Create a file called `fastapi_nginx` (like the one in this repository).

```bash
sudo vim /etc/nginx/sites-enabled/fastapi_nginx
```

And put this config into the file (replace the IP address with your EC2 instance's public IP):

```
server {
    listen 80;   
    server_name <YOUR_EC2_IP>;    
    location / {        
        proxy_pass http://127.0.0.1:8000;    
    }
}
```


Save the configuration

```
Ctrl + C --> :wq
```


Start NGINX.

```bash
sudo service nginx restart
```

Go to project folder

```bash
cd fastapi-tutorial
```

Create venv and install packages

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
```

Start the server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Update EC2 security-group settings for your instance to allow HTTP traffic to port 80.

Now when you visit your public IP of the instance, you should be able to access your API.

# Automate server start and stop using instance states

By using these commands your FastAPI application will turn on automatically whenever the instance is started and stops whenever the instance is stopped - Automation 101

Create a service

```bash
sudo nano /etc/systemd/system/fastapi.service
```

Add these configurations: 

```bash
[Unit]
Description=FastAPI Application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/v1-fast-api
ExecStart=/usr/bin/python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target

```

Reload the systemd daemon

```bash
sudo systemctl daemon-reload
```

Start the service immediately

```bash
sudo systemctl start fastapi.service
```

Enable the service to start on boot

```bash
sudo systemctl enable fastapi.service
```

Check the status

```bash
sudo systemctl status fastapi.service
```
