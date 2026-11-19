## My Docker Installation Guide

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


