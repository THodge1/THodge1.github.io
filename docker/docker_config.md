## My Docker Installation Guide

## Installation

First, I ran these commands to get it started and set-up.

```bash
$ sudo apt update && sudo apt install -y ca-certificates curl
```
This first one is a update and upgrade, adding two packages that allow easier and more secure data downloads
```bash
$ sudo install -m 0755 -d /etc/apt/keyrings
```
This command creates the folder that will store keys used to verify Docker
```bash
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc
```
This downloads the Docker public signing key and saves it. Key used to verify that the downloads came from the right source.
```bash
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list
```
Adds Docker's software repository to APT, so Ubuntu can find Docker when I update/install it.
```bash
$ sudo apt update
```
Updates the software to its most recent date.
```bash
$ sudo apt install -y docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
```
This installs Docker and its tools onto my VM.

*Spelling and syntax are crucial for the long commands^^*

## Testing

Then, I ran these to test the setup.

```bash
$ sudo systemctl enable --now docker
```
Starts Docker now and also makes Docker start when the VM boots.
```bash
$ systemctl is-enabled docker # enabled
```
Checks if Docker is enabled to boot when my VM does.
```bash
$ sudo docker run --rm hello-world
```
Runs Dockers small test container to check if it works.
```bash
$ docker compose version
```
Checks the version and displays for Docker compose.

## Installation of Gotify

For this, I began with the download command.
```bash
sudo docker run -d --name gotify --restart unless-stopped -p 8081:80 -v gotify-data:/app/data gotify/server
```
This command not only downloaded gotify using docker run, but also runs in the background, names the container gotify, and restarts on VM stoppage

After this, I ssh'd into my own console

I then ran:
```bash
curl "http://10.30.31.67/message?token=YOUR_APP_TOKEN" -F "title=Cyber Test" -F "message=Hello from my VM!" -F "priority=5"
```
This sent a message to my gotify to tell me that it is receiving.

## Securing the VM

For the requirements, I needed to have PasswordAuthentication no, and it was yes. I figured out that I needed to set a ssh key.

```bash
grep 'PasswordAuthentication ' /etc/ssh/sshd_config
```
Ran this to check passwordauthentication: it responded yes.


Now to fix this, I worked on my Mac terminal and proxmox.

```bash
ssh-keygen -t ed25519
```
I ran this to generate a ssh key.

```bash
cat ~/.ssh/id_ed25519.pub | ssh tah8872@10.30.31.67 'umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys'
```
I then ran this command on proxmox to copy the generated public key to my vm.

```bash
ssh -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 tah8872@10.30.31.67
```
I ran this to test the ssh in a new terminal tab.

```bash
sudo nano /etc/ssh/sshd_config
```
I went into the sshd_config file and changed the PasswordAuthentication to no.

```bash
sudo sshd -T | grep '^passwordauthentication'
```
I ran this to check it.

```bash
sudo grep -RnE '^[[:space:]]*(Include|Match|PasswordAuthentication|KbdInteractiveAuthentication)[[:space:]]' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/
```
I ran this to find anything that could be setting it to yes. I found that 50-cloud-init.conf was setting it to yes, and changed that file to no.

```bash
sudo ufw status verbose
```
I ran this to check the ufw firewall. It came back inactive meaning the firewall is inactive.

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```
I ran this to check the ports for gotify.

```bash
sudo ss -lntup
```
This shows the sockets, with letters that format the results into headings and includes tcp and udp.

```bash
sudo ufw allow 22/tcp
```
I ran this to allow the SSH connection from laptop to continue.

```bash
sudo ufw allow 22/tcp
```
I ran this to allow gotify's port connection.

```bash
sudo ufw default deny incoming
```
Ran this to block incoming connections unless specifically allowed.

```bash
sudo ufw default allow outgoing
```
This allows outgoing traffic, letting the VM initiate connections.

```bash
sudo ufw enable
```
Enables the firewall and allows it to enable at start up.

```bash
sudo ufw status numbered
```
This asks for the ufw status, and checks what configurations are on currently.

I then opened a new terminal tab and tested my ssh connection, and also used the hello world command above to test if gotify responds.

```bash
sudo apt upgrade && sudo apt install -y unattended-upgrades
```
This command allowed me to install unattended upgrades with -y being a automatic yes when prompted within the download.

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```
This enabled automatic updates within the unattended-upgrades.
A blue screen popped up asking if I wanted to automatically download and install updates. I chose yes.

```bash
sudo unattended-upgrade --dry-run --debug
```
This command let me run the feature without actually downloading anything.

## Backups and Restore Drill

First, I configure the script.

```bash
sudo docker inspect gotify --format '{{range .Mounts}}{{println .Type .Source "->" .Destination}}{{end}}'
```
This command found where the data is stored.

```bash
sudo mkdir -p /var/backups/gotify
```
This command made a new backup directory for the data in gotify.

```bash
sudo docker stop gotify
```
Stopped gotify so I can test the backup.

```bash
sudo tar -czf "/var/backups/gotify/gotify-$(date +%Y%m%d-%H%M%S).tar.gz" -C /var/lib/docker/volumes/gotify-data/_data .
```
I ran this to do a few things:
1. archive the files
2. specifies the archive files
3. gzips them

```bash
sudo docker start gotify
```
At this point start back up gotify.

```bash
sudo nano /usr/local/sbin/backup-gotify
```
This opens the backup file to begin the script.

```bash
set -euo pipefail
```
Tells the script to cancel if there are any errors.

```bash
backup_dir=/var/backups/gotify 
data_dir=/var/lib/docker/volumes/gotify-data/_data 
archive="$backup_dir/gotify-$(date +%Y%m%d-%H%M%S).tar.gz"
```
These are all made variables that makes it easier when writing out other functions later on.

```bash
trap 'docker start gotify >/dev/null' EXIT
```
This makes sure that if trying to start creates an error the script will exit.

```bash
if ! tar -czf "$archive" -C "$data_dir" .; then 
        rm -f "$archive"
        echo "Backup failed; Gotify will be restarted."
        exit 1 
fi
```
This is another error detection condition. Making sure the archive is created















