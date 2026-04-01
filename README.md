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
GRANT ALL PRIVILEGES ON my_app_db.* TO 'server-user'@'localhost';
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


### Enable Security (Can Connect to True (1))
-`sudo setsebool -P httpd_can_network_connect_db 1`

### Before Installing Git
- Update SSL: `sudo dnf update -y openssl`

### RHEL comes with git
- to configure git: `git config --global user.name "Firstname Lastname"`
- to configure: `git config --global user.email "your.email@example.com"`
- to clone reposotories: `cd` into desired directory then: `git clone https://github.com/username/repository-name.git`
- to open in VS code: `code .`


### Install nginx
- `sudo dnf install -y nginx`
- `sudo systemctl enable --now nginx`


### Create Self-signed cert
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/nginx-selfsigned.key \
  -out /etc/pki/tls/certs/nginx-selfsigned.crt \
  -subj "/CN=127.0.0.1"
```


### Configure the reverse proxy
- `sudo vi /etc/nginx/conf.d/api.conf`


```Nginx
server {
    listen 443 ssl;
    server_name 127.0.0.1;

    ssl_certificate /etc/pki/tls/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/pki/tls/private/nginx-selfsigned.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass https://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
- To save: esc then type :wq
- To quit: esc then type :q


### Adjust SELinux Policies
- `sudo setsebool -P httpd_can_network_connect 1`


### Test nginx config, Open Firewall, and Restart Nginx
- `sudo nginx -t`
- `sudo systemctl restart nginx`
- `sudo firewall-cmd --permanent --add-service=https`
- `sudo firewall-cmd --reload`


### Configure the DB layer of express.ts server
- NOTE: Use .env


```TypeScript
import mysql from 'mysql2/promise';

const pool = mysql.createPool({
  host: '127.0.0.1' || 'localhost', // Forces local loopback
  user: 'app_user',
  password: 'StrongPassword123!',
  database: 'app_db',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

export default pool;
```

*NOTE: Server id type is Number or int*


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
- `pm2 start dist/app.js --name "ts-api"`
- `pm2 save`


