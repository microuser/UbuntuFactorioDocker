# UbuntuFactorioDocker
For use with Ubuntu 18.10 and Docker to install Factorio

This assumes a newly formatted KVM install of Ubuntu18.10

## Update
```sh 
sudo apt-get update
sudo apt-get upgrade
```
On my VPS which is a KVM linux kernel, 
I replaced boot loader on the MBR of the disk
I also choose to keep the existing SSH file.

## Install Docker
```sh
sudo apt-get -y install curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
apt-cache policy docker-ce
sudo apt-get install -y docker-ce
sudo systemctl status docker
```

## Add a ssh user (Optional)
```sh
user=user #set your username here, on the right side of the equals
adduser $user
usermod -aG sudo $user
su - $user
```
## Add docker group to your current user
Add yourself to the docker group
```sh
sudo usermod -aG docker ${USER}
```
Update your current user's groups for this session. #this starts a session with the new docker permissions, preventing a restart/logout
```sh
sudo su - ${USER} 
```
List the groups you are a part of
```sh
id -nG
```

##Get the Factorio Docker
as a user with docker group permissions (from above)
```sh
docker pull factoriotools/factorio
```
