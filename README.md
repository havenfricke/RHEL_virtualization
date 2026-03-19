### Generic System Packages Update
- `sudo dnf update -y`

### Verify CPU Supports Virtualization 

- `lscpu | grep Virtualization`
- If "VT-x" (Intel) or AMD-V (AMD) returned, virtualization is supported 


### Install Virtualization Packages

- Install the core virtualization group with: `sudo dnf groupinstall "Virtualization Host" -y`
- Install additional managment tools: `sudo dnf virt-install virt-viewer -y`


### Install Cockpit and VM Module

- `sudo dnf install cockpit-machines -y`


### Spinning Up the Services

- Start and enable libvirt: `sudo systemctl enable --now libvirtd`
- Start and enable Cockpit: `sudo systemctl enable --now cockpit.socket`

Cockpit uses port 9090

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

    Click Virtual Machines in the left sidebar.

    Select Create VM.

    Name: Enter a unique identifier for the VM.

    Installation Type: Choose "Local Install Media" (ISO), "URL," or "Cloud Image."

    Operating System: Select the OS type (this optimizes hardware settings).

    Storage: Allocate disk space (e.g., 20 GiB).

    Memory: Set the RAM (e.g., 2048 MiB).

    Click Create. The VM will appear in the list where you can start it and access the VNC console.


### Creating a VM via virt-install (CLI)

The CLI is preferred for automation and specific hardware passthrough. Use the following template for a standard RHEL-based VM:

```Bash
sudo virt-install \
--name rhel-guest \
--vcpus 2 \
--memory 2048 \
--disk path=/var/lib/libvirt/images/rhel-guest.qcow2,size=20 \
--network network=default \
--os-variant rhel9.0 \
--cdrom /path/to/your/rhel-installer.iso \
--graphics vnc
```

### RHEL comes with git
- to configure git: `git config --global user.name "Firstname Lastname"`
- to configure: `git config --global user.email "your.email@example.com"`

### MySQL Install
- `sudo dnf install mysql8.4-server` (or latest version)

### Offline Token
eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI0NzQzYTkzMC03YmJiLTRkZGQtOTgzMS00ODcxNGRlZDc0YjUifQ.eyJpYXQiOjE3NzM4OTYwNjMsImp0aSI6ImQ1NTQ0MTAwLTQ3MGItNGY3Ni1hY2M3LTRiNGE2ODhmNzYyNiIsImlzcyI6Imh0dHBzOi8vc3NvLnJlZGhhdC5jb20vYXV0aC9yZWFsbXMvcmVkaGF0LWV4dGVybmFsIiwiYXVkIjoiaHR0cHM6Ly9zc28ucmVkaGF0LmNvbS9hdXRoL3JlYWxtcy9yZWRoYXQtZXh0ZXJuYWwiLCJzdWIiOiJmOjUyOGQ3NmZmLWY3MDgtNDNlZC04Y2Q1LWZlMTZmNGZlMGNlNjpoYXZlbmZyaWNrZUB1LmJvaXNlc3RhdGUuZWR1IiwidHlwIjoiT2ZmbGluZSIsImF6cCI6InJoc20tYXBpIiwic2lkIjoiYWM4NWViMDItMWE2MC00YjUyLTkxYzAtNTE0MjhiOTM2NmQ3Iiwic2NvcGUiOiJiYXNpYyByb2xlcyB3ZWItb3JpZ2lucyBjbGllbnRfdHlwZS5wcmVfa2MyNSBvZmZsaW5lX2FjY2VzcyJ9.vUr1dpF7cO-c60cCN3EBGQxGOHacM2Fz8qeue7ulujFNYNHCICvrBrCQ0H-zEQOTx8SnKe79llMHfZk2rQNVUw