# Docker_Portainer_setup
Steps to install Docker and Portainer-CE on CentOS 7.  

# Install Docker-CE
There are many other places that describe how to install Docker-CE on CentOS 7.  In it's simplest way, you need to install the Docker Repo found here:  
https://docs.docker.com/engine/install/centos/

`sudo yum install yum-utils`  
`sudo yum install yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`  

This could have been done by just downloading the docker-ce.repo file to the /etc/yum.repo.d/ folder:  
`sudo wget -o /etc/yum.repo.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo`

Then install docker-ce (this will also install docker-ce-cli & containerd.io):  
`sudo yum install docker-ce`  

# Add your user to the group "docker"
`sudo useradd -aG docker YOURUSERNAME`  

# Install Registry so we can have our own local registry (nice for security):
For more details see:  
https://docs.docker.com/registry/deploying/

`docker pull registry:2.8.1`  
`docker volume create registry_data`  
`docker run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.8.1`  

You may need to allow "insecure-registries" if you are just testing locally or in an air-gapped environment.  
https://docs.docker.com/registry/insecure/  
For Podman installations: `podman push --tls-verify=false localhost:5000/my-special-container` should work fine for pushes and pulls locally.

# Pull-tag-push portainer/portainer-ce
`docker pull portainer/portainer-ce:linux-amd64-2.11.1`  
`docker tag portainer/portainer-ce:linux-amd64-2.11.1 localhost:5000/portainer-ce:linux-amd64-2.11.1`  
`docker push localhost:5000/portainer-ce:linux-amd64-2.11.1`  

# Create a volume and spin up Portainer-CE
`docker volume create portainer_data`  
`docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data localhost:5000/portainer-ce:linux-amd64-2.11.1`  
NOTE: For Podman, you'll need to start the podman socket first.  `sudo systemctl enable --now podman.socket`  And you will need to run the portainer-ce container as root :(  

You can include a password from a password file -v password_file:/tmp/portainer_password  
Details here:
https://documentation.portainer.io/v2.0/deploy/cli/
`docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_password:/tmp/portainer_password localhost:5000/portainer-ce:linux-amd64-2.11.1 --admin-password-file /tmp/portainer_password`

If you do NOT use a password file, the first time you bring up the page it will have you create an admin account and password.  
Login to the new Portainer-CE web site:  
http://localhost:9000/  
Create the admin account and password.

Click on the left button and confirm you have linked the /var/run/docker.sock  
You may want to add the local registry.  More on that later.
