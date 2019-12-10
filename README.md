# UbuntuFactorioDocker
For use with Ubuntu 18.10 and Docker to install Factorio

This assumes a newly formatted KVM install of Ubuntu18.10

## Update
```sh 
sudo apt-get update ;
sudo apt-get -y upgrade ;
```
On my VPS which is a KVM linux kernel, 
I replaced boot loader on the MBR of the disk
I also choose to keep the existing SSH file.

## Install Docker
```sh
sudo apt-get -y install curl software-properties-common ;
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - ;
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" ;
sudo apt-get update ; 
apt-cache policy docker-ce ;
sudo apt-get install -y docker-ce ;
sudo systemctl status docker ;
```

## Add a ssh user (Optional)
```sh
user=user #set your username here, on the right side of the equals ;
adduser $user ;
usermod -aG sudo $user ;
su - $user ;
```
## Add docker group to your current user
Add yourself to the docker group
```sh
sudo usermod -aG docker ${USER} ;
```
Update your current user's groups for this session. #this starts a session with the new docker permissions, preventing a restart/logout
```sh
sudo su - ${USER} ;
```
List the groups you are a part of
```sh
id -nG ;
```

##Get the Factorio Docker
as a user with docker group permissions (from above)
```sh
docker pull factoriotools/factorio ;
```
The shared location for user executiable files (opt)
```sh
sudo mkdir -p /opt/factorio ;
```
set folder permissions
```sh 
sudo chown 845:845 /opt/factorio ;
```

in order to troubleshoot, lets start docker in an interactive console first
```sh
docker run -it \
  -p 34197:34197/udp \
  -p 27015:27015/tcp \
  -v /opt/factorio:/factorio \
  --name factorio \
  factoriotools/factorio ;
```

or if you want to inspect the files at a console itself
```sh
docker exec -it factorio sh ; 
```

if it drops you from the console, perhaps you want to ```docker attach factorio```


run factorio in daemon (background) mode and port forward
```sh
docker run -d \
  -p 34197:34197/udp \
  -p 27015:27015/tcp \
  -v /opt/factorio:/factorio \
  --name factorio \
  --restart=always \
  factoriotools/factorio ;
```
Did you make a mistake? perhaps to start again you want to 
```sh
#echo DANGER. WILL DESTROY ALL DOCKER DATA
#docker rm $(docker ps -a -q) -f ;
```
Check the logs. We should see some errors, cause we have an invalid starting config
```sh
docker logs factorio ;
```
Stop factorio
```sh
docker stop factorio ;
```

Lets edit the config file
```sh
sudo nano /opt/factorio/config/server-settings.json ;
```
In the above edit, I changed the following
1.  
```json
"visibility":
  {
    "public": false,
    "lan": true
  },
```
2. 
```json
"require_user_verification": false,
```
3. 
```json
"only_admins_can_pause_the_game": false,
```
4. 
```json
 "autosave_only_on_server": false,
```

Since I used nano as the text editor, I press Ctrl-X to save and exit

##
Run Factorio
Clear Logs
```sh
sudo sh -c "truncate -s 0 /var/lib/docker/containers/*/*-json.log" ;
docker logs factorio ;
```
restart and inspect logs
```sh
docker stop factorio ;
docker start factorio ;
docker logs factorio ;
```

Interactive Commands
```sh
docker run -d -it  \
      --name factorio \
      factoriotools/factorio
docker attach factorio ;

```



##
Server Settings which work with public server
```json
{
  "name": "Alliteration Alliance",
  "description": "Saucy Savages Survival",
  "tags": ["game", "tags"],

  "_comment_max_players": "Maximum number of players allowed, admins can join even a full server. 0 means unlimited.",
  "max_players": 0,

  "_comment_visibility": ["public: Game will be published on the official Factorio matching server",
                          "lan: Game will be broadcast on LAN"],
  "visibility":
  {
    "public": true,
    "lan": true
  },

  "_comment_credentials": "Your factorio.com login credentials. Required for games with visibility public",
  "username": "USE_YOUR_OWN_USERNAME",
  "password": "",

  "_comment_token": "Authentication token. May be used instead of 'password' above.",
  "token": "LOG ON TO THE FACTORIO WEBSITE AND GET YOUR TOKEN",

  "game_password": "",

  "_comment_require_user_verification": "When set to true, the server will only allow clients that have a valid Factorio.com account",
  "require_user_verification": false,

  "_comment_max_upload_in_kilobytes_per_second" : "optional, default value is 0. 0 means unlimited.",
  "max_upload_in_kilobytes_per_second": 0,

  "_comment_max_upload_slots" : "optional, default value is 5. 0 means unlimited.",
  "max_upload_slots": 5,

  "_comment_minimum_latency_in_ticks": "optional one tick is 16ms in default speed, default value is 0. 0 means no minimum.",
  "minimum_latency_in_ticks": 0,

  "_comment_ignore_player_limit_for_returning_players": "Players that played on this map already can join even when the max player limit was reached.",
  "ignore_player_limit_for_returning_players": false,

  "_comment_allow_commands": "possible values are, true, false and admins-only",
  "allow_commands": "admins-only",

  "_comment_autosave_interval": "Autosave interval in minutes",
  "autosave_interval": 10,

  "_comment_autosave_slots": "server autosave slots, it is cycled through when the server autosaves.",
  "autosave_slots": 5,

  "_comment_afk_autokick_interval": "How many minutes until someone is kicked when doing nothing, 0 for never.",
  "afk_autokick_interval": 0,

  "_comment_auto_pause": "Whether should the server be paused when no players are present.",
  "auto_pause": true,

  "only_admins_can_pause_the_game": false,

  "_comment_autosave_only_on_server": "Whether autosaves should be saved only on server or also on all connected clients. Default is true.",
  "autosave_only_on_server": false,

  "_comment_non_blocking_saving": "Highly experimental feature, enable only at your own risk of losing your saves. On UNIX systems, server will fork itself to create an autosave. Autosaving on connected Windows clients will be disabled regardless of autosave_only_on_server option.",
  "non_blocking_saving": false,

  "_comment_segment_sizes": "Long network messages are split into segments that are sent over multiple ticks. Their size depends on the number of peers currently connected. Increasing the segment size will increase upload bandwidth requirement for the server and download bandwidth requirement for clients. This setting only affects server outbound messages. Changing these settings can have a negative impact on connection stability for some clients.",
  "minimum_segment_size": 25,
  "minimum_segment_size_peer_count": 20,
  "maximum_segment_size": 100,
  "maximum_segment_size_peer_count": 10
}

```


## More Troubleshooting
Sometimes there are issues with the firewall

```
sudo ufw allow 34197/udp
sudo ufw status verbose
sudo ufw allow ssh/tcp
sudo ufw enable
sudo ufw status
```

### Disable IPV6
open editor
```sh
sudo nano /etc/sysctl.conf
```
append the following to the end
```txt
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 =1
```
restart kernel
```sh
sudo sysctl -p
```
verify 
```sh
cat /proc/sys/net/ipv6/conf/all/disable_ipv6 
```

### Override the DNS 
```sh
sudo nano /etc/dhcp/dhclient.conf
```
append the text to use google's dns servers
```txt
supersede domain-name-servers 8.8.8.8, 8.8.4.4

interface "eth0" {
        prepend domain-name-servers 8.8.8.8
}

```
