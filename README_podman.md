WIP

# Podman_Portainer_setup
Steps to install Podman and Portainer-CE on Fedora 35.  In order to run Portainer under Podman, you need to run the container under a
Tmux or Screen session.  This container unfortunately needs to run as root.  

# Install Podman
There are many other places that describe how to install Podman on Fedora.  In addition to Podman, I recommend also installing JQ and OpenSSL.  

`sudo yum install podman jq openssl tmux`   

# Some more steps are needed before this, but here is how you run the container.
## NOTE: runs as root, with `--privilege` and with `:z` on the socket `-v`.  Also, need to start the Podman service:
`sudo systemctl enable podman.sock` and `sudo systemctl start podman.sock`  
`sudo podman run --rm -d -p 9000:9000 --privileged --name portainer -v /run/podman/podman.sock:/var/run/docker.sock:z -v portainer_data:/data localhost:5000/portainer-ce:linux-amd64-2.11.1`

# Install Registry so we can have our own local registry (nice for security):
For more details see:  
https://docs.docker.com/registry/deploying/

`podman pull registry:2.8.1`  
`podman volume create registry_data`  
`podman run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.8.1`  

You may need to allow "insecure-registries" if you are just testing locally or in an air-gapped environment (non-https).  
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
