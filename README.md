### Generic System Packages Update
- `sudo dnf update -y`

### Verify CPU Supports Virtualization 

- `lscpu | grep Virtualization`
- If "VT-x" (Intel) or AMD-V (AMD) returned, virtualization is supported 


### Install Virtualization Packages
- Install the core virtualization group with: `sudo dnf groupinstall "Virtualization Host" -y`
- Install additional managment tools: `sudo dnf virt-install virt-viewer -y`

OR

- `sudo dnf install qemu-kvm libvirt virt-install virt-viewer`


### Install Cockpit and VM Module
- `sudo dnf install cockpit-machines -y`


### Spinning Up the Services

- Start and enable libvirt: `sudo systemctl enable --now libvirtd`
- Start and enable Cockpit: `sudo systemctl enable --now cockpit.socket`

*Cockpit uses port 9090 (127.0.0.1:9090)*

- Open firewall for cockpit: `sudo firewall-cmd --add-service=cockpit --permanent` 
- Reload firewall `sudo firewall-cmd --reload`

- Add user to the libvirt group (optional, for non-root management): `sudo usermod -aG libvirt $(whoami)` (otherwise use root user credentials to log in to Cockpit)


### Access Web Console

- Open web browser and navigate to https://<your-server-ip>:9090 or https://localhost:9090
- Log in with RHEL system credentials.

- Click on Virtual Machines in the left-hand sidebar.

Note: If the "Virtual Machines" tab is missing, ensure cockpit-machines was installed correctly and refresh the page.


### Verify KVM Modules Loaded

- `lsmod | grep kvm`


### Creating a VM via Cockpit (GUI)

The Cockpit web interface is the most visual way to manage VMs. Once logged in at https://<server-ip>:9090:

- Click Virtual Machines in the left sidebar.

- Select Create VM.

- Name: Enter a unique identifier for the VM.

- Installation Type: Choose "Local Install Media" (ISO), "URL," or "Cloud Image."

- Operating System: Select the OS type (this optimizes hardware settings).

- Storage: Allocate disk space (e.g., 20 GiB).

- Memory: Set the RAM (e.g., 2048 MiB).

- Click Create. The VM will appear in the list where you can start it and access the VNC console.


### Creating a VM via virt-install (CLI)

The CLI is preferred for automation and specific hardware passthrough. Use the following template for a standard RHEL-based VM:
(Needs more documentation)

```Bash
sudo virt-install \
--name rhel-guest \
--vcpus 2 \
--memory 2048 \
--disk path=/var/lib/libvirt/images/rhel-guest.qcow2,size=20 \
--network network=default \
--os-variant rhel10.0 \
--cdrom /path/to/your/rhel-installer.iso \
--graphics vnc
```

**NOTE: Everything after this is done inside the virtual machine instance**


### MySQL Install
- `sudo dnf install mysql8.4-server` (or latest version)

### MySQL Enable
- `sudo systemctl enable mysqld`

### MySQL Start
- `sudo systemctl start mysqld`

### MySQL Secure Setup
- `sudo mysql_secure_installation`
- This will prompt the user for configurations. Complete the setup and installation before the next step.

### MySQL Shell Login (USE THIS TO RUN SQL COMMANDS)
- `sudo mysql -u root -p`

### Create Database and Server Access
```SQL
CREATE DATABASE mainDB;
CREATE USER 'server-user'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON mainDB.* TO 'server-user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Drop database and created user (reverse above action)
```SQL
DROP USER 'app_user'@'127.0.0.1';
DROP DATABASE mainDB;`
```

### Use Database
- `USE db_name;`

### Enable DB Connection (Can Connect to True (1))
-`sudo setsebool -P httpd_can_network_connect_db 1`

*The following git commands may prompt you to install git. If so, install it.*

### Before Installing Git
- Update SSL: `sudo dnf update -y openssl`

### Git config set up
- to configure git: `git config --global user.name "Firstname Lastname"`
- to configure: `git config --global user.email "your.email@example.com"`
- to clone reposotories: `cd` into desired directory then: `git clone https://github.com/username/repository-name.git`
- to open in VS code: `code .`

### Configure the DB layer of [python_webserver](https://github.com/havenfricke/python_webserver) server via

```Python
import os
from dotenv import load_dotenv
from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine

load_dotenv()

DB_USER = os.getenv('DB_USER')
DB_PASS = os.getenv('DB_PASS')
DB_HOST = os.getenv('DB_HOST')
DB_NAME = os.getenv('DB_NAME')

DB_URL = f"mysql+aiomysql://{DB_USER}:{DB_PASS}@{DB_HOST}/{DB_NAME}"

engine = create_async_engine(DB_URL, pool_pre_ping=True)
```

### Before Installing Node.js
- Update SSL: `sudo dnf update -y openssl`

### Install Node.js
- `sudo dnf install nodejs -y`
- Verify node.js install: `node -v `
- Verify node package manager install: `npm -v`

### Build the TS Server
- `npm run build` (npx tsc with package.json)

### Install PM2 and run server
- `sudo npm install -g pm2`
- `pm2 start dist/app.js --name "ts-api"` or `pm2 start main.py --name "py-api"`
- `pm2 save`


*To obtain SSL certificates for the paths specified in your Nginx configuration, the industry standard is to use Certbot, a free tool provided by the Electronic Frontier Foundation (EFF) that interfaces with the Let's Encrypt Certificate Authority.*

[Let's Encrypt](https://letsencrypt.org/)


### Install nginx
- `sudo dnf install -y nginx`
- `sudo systemctl enable --now nginx`

### RHEL 10 requires the EPEL repository to access Certbot.
```Bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
sudo dnf upgrade
```

### Install Certbot
- `sudo dnf install certbot python3-certbot-nginx`

### Run certbot
- `sudo certbot --nginx -d yourdomain.com`

### Verify Auto-Renewal
- `sudo certbot renew --dry-run`

### Create cert directory
- `sudo mkdir -p /etc/letsencrypt/live/yourdomain.com/`

### Generate keys in created directory
```Bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/letsencrypt/live/yourdomain.com/privkey.pem \
-out /etc/letsencrypt/live/yourdomain.com/fullchain.pem
```

### Update the security context of the certificate files
- `sudo chcon -t cert_t /etc/letsencrypt/live/yourdomain.com/*.pem`

### Enable http protocol to connect to the network (Can Connect to True (1))
- `sudo setsebool -P httpd_can_network_connect 1`

```Bash
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload`
```

### Use vim to edit nginx config file
- `sudo vim /etc/nginx/nginx.conf`

```Nginx
# Define the pool of backend servers
upstream fastapi_cluster {
    server 127.0.0.1:8000;
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    listen 80;
    server_name yourdomain.com;
    
    # Redirect all HTTP requests to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    # SSL Configuration (Use Certbot for automated setup)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        # Direct traffic to the upstream block named 'fastapi_cluster'
        proxy_pass http://fastapi_cluster;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        
        # Required headers for FastAPI/Uvicorn to accurately identify the client IP and HTTPS scheme
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
- To quit and save: esc or ctrl + c then type :wq
- To quit without save: esc or ctrl + c then type :q


### Open Firewall, Test nginx config, and Restart Nginx
- `sudo firewall-cmd --permanent --add-service=https`
- `sudo firewall-cmd --reload`
- `sudo nginx -t`
- `sudo systemctl restart nginx`

### To load balance pm2 instances and match nginx config
```Bash
for port in {8000..8003}; do
  pm2 start "python3 -m uvicorn main:app --host 127.0.0.1 --port $port" --name "my-api-$port";
done
```
### Write the current process configuration to a dump file that the PM2 daemon will read upon the next boot
- `pm2 startup`generates a custom command in the terminal output. 
- Copy and paste that specific output into the terminal and execute it to register PM2
- Once instances are online and running: `pm2 save` writes process configuration to file PM2 daemon will read next boot

### How PM2 and Nginx Work Together
When using Nginx as a reverse proxy with multiple Python instances, Nginx must be configured with an upstream block that points to the specific local ports assigned to each PM2 instance (e.g., 8000, 8001, 8002).

**Nginx**: Receives the public HTTPS request and acts as a load balancer, distributing traffic across the defined backend ports in the upstream pool.

**PM2**: Operates in fork mode, managing completely separate and independent Python processes rather than sharing a single port.

**Instances**: Each Uvicorn worker process is explicitly bound to its own unique port and handles the routed requests independently.

### Check pm2 status
- Check Status: `pm2 list` (You will see multiple rows for the same app name).
- Monitor resources: `pm2 monit` to ensure instances aren't competing for memory.

### Important considerations
- **Statelessness**: Since requests are distributed, you cannot store user sessions in local memory (variables). Use a database for session management.
- **Port Sharing**: Do not try to assign different ports to different instances. PM2 handles the port sharing internally on the single port your app is configured to use (e.g., 3000).
