# Docker_Portainer_setup
Steps to install Docker and Portainer-CE on CentOS 7.  

# Install Docker-CE
There are many other places that describe how to install Docker-CE on CentOS 7.  In it's simplest way, you need to install the Docker Repo found here:  
https://docs.docker.com/engine/install/centos/

`sudo yum install yum-utils`  
`sudo yum install yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`  

This could have been down by just downloading the docker-ce.repo file to the /etc/yum.repo.d/ folder:  
`sudo wget -o /etc/yum.repo.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo`

Then install docker-ce (this will also install docker-ce-cli & containerd.io):  
`sudo yum install docker-ce`

# Add your user to the group "docker"
`sudo useradd -aG docker YOURUSERNAME`

# Install Registry so we can have our own local registry (nice for security):
For more details see:  
https://docs.docker.com/registry/deploying/

`docker pull registry:2.7.1`
`docker volume create registry_data`
`docker run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.7.1`

# Pull portainer/portainer-ce
`docker pull portainer/portainer-ce:2.0.0`
`docker tag portainer/portainer-ce:2.0.0 localhost:5000/portainer-ce:2.0.0`

# Create a volume and spin up Portainer-CE
`docker volume create portainer_data`
`docker run -d --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data localhost:5000/portainer-ce:2.0.0`

You can include a password from a password file -v password_file:/tmp/portainer_password  
Details here:
https://documentation.portainer.io/v2.0/deploy/cli/
`docker run -d --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_password:/tmp/portainer_password localhost:5000/portainer-ce:2.0.0 --admin-password-file /tmp/portainer_password`

If you do NOT use a password file, the first time you bring up the page it will have you create an admin account and password.  
Login to the new Portainer-CE web site:  
http://localhost:5000/  
Create the admin account and password.

Click on the left button and confirm you have linked the /var/run/docker.sock
You may want to add the local registry.  More on that later.
