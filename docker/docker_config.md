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













