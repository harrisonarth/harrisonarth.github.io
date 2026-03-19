+++
title = "Installing Docker"
date = 2026-03-13
tags = ['bash', 'docker']
+++

Download and run the convenience script from Docker:
 
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
 
This script auto-detects your distro, adds Docker's official repository, and installs docker-ce, docker-ce-cli, containerd.io, as well as compose and buildx plugins. It works on Ubuntu, Debian, Fedora, and most common distros. Always review scripts before piping them into sh.
 
### Verify the Daemon Is Running
 
```bash
sudo systemctl status docker
```
 
You should see `active (running)`. If not:
 
```bash
sudo systemctl start docker
sudo systemctl enable docker
```
 
`enable` ensures Docker starts automatically on boot.
 
### Run Docker Without Sudo
 
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
 
`groupadd docker` creates the docker group if it doesn't already exist (the install script may have already done this, the command is harmless if the group is already there). 

`usermod -aG` appends your user to it without affecting your other groups. 

`newgrp` activates the membership in your current shell. Alternatively, log out and back in.
 
> **Security note:** Members of the `docker` group have root-equivalent access on the host, since they can mount any filesystem into a container. Only add trusted users.
 
### Verify It Works
 
```bash
docker run hello-world
docker compose version
```
 
Both should work without `sudo`. If `docker run` gives a permission error, try logging out and back in so the group membership takes full effect.
 
Remove the install script once you're done:
 
```bash
rm get-docker.sh
```