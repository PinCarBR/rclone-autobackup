# rclone-autobackup
This repository can be used to create an auto backup feature to sync your local folder to the cloud using rclone.
It is pre-configured to work with Google Cloud storage but it can be adjusted to work with any provider supported by rclone. Please check rclone documentation page at https://rclone.org to configure for your own cloud storage service.
## Minimum VPS configuration
- OS: Ubuntu 16.04 x64
- RAM: 512 MB
- CPU: 1
- Disk space: 10 GB
## Setup Instructions
### Install Docker
1. First, in order to ensure the downloads are valid, add the GPG key for the official Docker repository to your system:  
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
2. Add the Docker repository to APT sources:  
`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
3. Next, update the package database with the Docker packages from the newly added repo:  
`sudo apt-get update`
4. Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:  
`apt-cache policy docker-ce`  
 You should see output similar to the follow:  
```
docker-ce:
    Candidate: 18.06.1~ce~3-0~ubuntu
    Version table:
        18.06.1~ce~3-0~ubuntu 500
            500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```
5. Finally, install Docker:  
`sudo apt-get install -y docker-ce`
6. Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it's running:  
`sudo systemctl status docker`
7. If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:  
`sudo usermod -aG docker ${USER}`
8. To apply the new group membership, log out of the server and back in.
9. Afterwards, you can confirm that your user is now added to the docker group by typing:  
`id -nG`  

- For more info, visit: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04
### Install Docker Compose
1. We'll check the current release and if necessary, update it in the command below:  
`ssudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. Next we'll set the permissions:  
`sudo chmod +x /usr/local/bin/docker-compose`
3. Then we'll verify that the installation was successful by checking the version:  
`docker-compose -v`

- For more info, visit: https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-16-04
### Install the repository into the VPS
1. Clone the repository.
1. Step into the just created folder 
### Configure environment variables
The file `.sample_env` has a sample set of environmental variables that needs to be configured. The sample provided are required for google cloud storage and you need to add the environment variables required by your cloud storage provider, please check documentation page at https://rclone.org to configure for your own cloud storage service.
On top of the environment variables for the cloud storage service, you need to configure the following environment variables for the scripts to work:
```
REMOTE_NAME=MYGS (example)
BUCKET_NAME=your-bucket-name
LOCAL_SYNC_FOLDER=/opt/mysql/backup (example)
FREQUENCY=daily
```
**You need to create a file named `.env` with your own configuration.**
### Load your authentication file - Google cloud storage only
1. Create a service account key by following the instructions at https://cloud.google.com/iam/docs/creating-managing-service-account-keys#creating_service_account_keys
2. Copy the key to the folder `/auth` with the name `key.json`  
**If your cloud service provider works also with authentication keys, this may work in the same way, if not, the docker-compose file and maybe the scripts need to be adapted to your provider requirements.**
### Execute all the services
`docker-compose up -d` or `docker-compose up -d --build --force-recreate` to force recreation of image and container  
Check if everything is working via the commands:  
`docker-compose ps` or `docker ps`  
Check the logs of each container via the command:  
`docker-compose logs container_name`
### APPLICATION NOTES
1. When the container is started, it will consider the information on the cloud as the single source of truth, i.e.: it will sync the local folder with the information in the cloud and erase any file in the local folder that is not in the cloud.  
**CAUTION: if you have files in your local folder that are not in the cloud, they will be ERASED during this step!!!**
2. Once the container is started, it will sync the cloud bucket defined on the parameter `BUCKET_NAME` with the content of the local folder defined with the parameter `LOCAL_SYNC_FOLDER` at the frequency defined with the parameter `FREQUENCY` in your `.env` file.  
**CAUTION: if you delete a file in your local folder, they will also be deleted in the cloud during this step!!!**
