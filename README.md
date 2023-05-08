# Deploy FastAPI Applications with Gunicorn and Nginx on Ubuntu 22.04

- #### May 8, 2023
‍‍‍
- ### In this tutorial, we used the root user to deploy. Our suggestion is to use a non-root user for security reasons.


<br><br>

1. ## Update Your Server 
```lang=bash
apt update && apt upgrade -y
```

<br>


2. ## Initialize Project Directory

<br>

#### Create a folder for your project

```lang=bash
mkdir /root/fastapi_project
```

<br>


#### Open the project directory

```lang=bash
cd /root/fastapi_project
```

<br>


3. ## Create Virtual Environment

#### Install Python virtual environment package

```lang=bash
apt install python3-venv
```

<br>


#### Create a Python virtual environment for your project


```lang=bash
python3 -m venv venv 
```

<br>


#### Enter the virtual environment

```lang=bash
source venv/bin/activate
```

<br>


#### Exit the virtual environment

```lang=bash
deactivate
```

<br>


3. ## Install Dependencies

<br>


#### Install wheel Python package

```lang=bash
pip install wheel
```

<br>


#### install your project requirements

```lang=bash
pip install -r requirements.txt
```

<br>



#### Install Gunicorn

```lang=bash
pip install gunicorn
```

<br>


### Create Systemd Service


#### Add configuration file for Gunicorn

```lang=bash
nano conf.py
```

<br>


#### Paste the following code and save the file using CTRL + X then ENTER


```lang=bash
from multiprocessing import cpu_count

bind = "127.0.0.1:8000"

# Worker Options
workers = cpu_count() + 1
worker_class = 'uvicorn.workers.UvicornWorker'

# Logging Options
loglevel = 'debug'
accesslog = '/root/fastapi_project/access_log'
errorlog =  '/root/fastapi_project/error_log'
```

<br>


### Create and edit systemd unit file


```lang=bash
nano /etc/systemd/system/fastapi.service
```

<br>

#### Paste the following code and save the file using CTRL + X then ENTER

```lang=bash
[Unit]

Description=Gunicorn Daemon for FastAPI Application

After=network.target



[Service]

User=root

Group=www-data

WorkingDirectory=/root/fastapi_project

ExecStart=/root/fastapi_project/venv/bin/gunicorn -c conf.py main:app



[Install]

WantedBy=multi-user.target
```


<br>


### Enable and start the service

```lang=bash
systemctl enable --now fastapi
```

<br>


### Start & stop & disable & restart the service

```lang=bash
systemctl start fastapi

systemctl stop fastapi

systemctl disable fastapi

systemctl restart fastapi
```

<br>


### To verify if everything works run the following command

```lang=bash
systemctl status fastapi
```

<br><br>



4. ## Setup Nginx as Reverse Proxy


#### Install Nginx

```lang=bash
apt install nginx -y
```

<br>


#### Create Certificate for SSL with OpenSSL 

`replace domain.com with your actual domain`

```lang=bash
openssl req -new -newkey rsa:4096 -days 735 -nodes -x509 -subj '/C=UK/ST=Denial/L=String/O=Dis/CN=domain.com' -keyout /etc/ssl/domain_com.key -out /etc/ssl/domain_com.cert
```

<br>

#### Add vhost configuration

Add vhost file to `sites-available` directory


```lang=bash
nano /etc/nginx/sites-available/fastapi_project
```

<br><br>

#### Paste the following content (replace your_domain with your actual domain) and save the file using CTRL + X then ENTER

`replace domain_com with your actual domain`

```lang=bash
server {

	client_max_body_size 64M;

        listen 443 ssl;
        listen [::]:443 ssl;

        ssl_certificate    /etc/ssl/domain_com.cert;
        ssl_certificate_key    /etc/ssl/domain_com.key;


        server_name api.domain.com; # Replace Your Domain


        location / {
                proxy_pass             http://127.0.0.1:8000;
                proxy_read_timeout     60;
                proxy_connect_timeout  60;
                proxy_redirect         off;

                # Allow the use of websockets
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        
	}
}

```


<br>

#### Activate vhost configuration

- Add a `soft link` of the vhost file in the `sites-enabled` directory


```lang=bash
ln -s /etc/nginx/sites-available/fastapi_project /etc/nginx/sites-enabled/
```

<br>

#### Test the configuration

```lang=bash
nginx -t
```

<br>

#### Expected output:

```lang=bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful
```

<br>

#### Reload Nginx

```lang=bash
systemctl reload nginx
```

<br>




### Original source
- [Deploying FastAPI on Ubuntu with Nginx and Let’s Encrypt
](https://www.slingacademy.com/article/deploying-fastapi-on-ubuntu-with-nginx-and-lets-encrypt/)
