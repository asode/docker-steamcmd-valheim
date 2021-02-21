# Set up Valheim dedicated server with steamcmd using docker

## About The Project

* Just the basics to setup a dedicated server for Valheim
* No guarantees it will work
* No idea what I'm doing, use at your own risk
* I used firewalld, but you can secure your server any way you want
* Tested with Fedora 33
* Inspired by <a target="_blank" href="https://github.com/Nimdy/Dedicated_Valheim_Server_Script">Nimdy's Dedicated_Valheim_Server_Script</a>

## Prerequisites

* Set up a server
* Create and use a different user than root <b>(Optional)</b>
* Set up SSH
  * to use a different port <b>(Optional)</b>
  * force users to login with a SSH key, up to you <b>(Optional)</b>
* <a target="_blank" href="https://docs.docker.com/engine/install/">Install Docker</a>
  ```
  # For Fedora
  sudo dnf -y install dnf-plugins-core
  sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
  sudo dnf install docker-ce docker-ce-cli containerd.io
  ```
* Open ports used by Steam and Valheim
  ```
  # You might need to install firewalld first
  sudo dnf install firewalld -y
  # Start firewalld
  sudo systemctl start firewalld
  # To make firewalld start automatically at system start:
  sudo systemctl enable firewalld
  
  # Add needed ports
  # Valheim
  sudo firewall-cmd --add-port=2456-2458/tcp --permanent
  sudo firewall-cmd --add-port=2456-2458/udp --permanent
  # Steam
  sudo firewall-cmd --add-port=27015/tcp --permanent
  sudo firewall-cmd --add-port=27015/udp --permanent
  
  sudo firewall-cmd --reload
  ```
  
## Down to business

* Start steamcmd using docker image
  * Map directories with ```-v``` switch
    * You can find your world files in ```.config/unity3d/IronGate/Valheim```
  * Expose ports from docker image with ```-p``` switch
  ```
  sudo docker run -it --name=steamcmd \
  -v ~/steam/valheimserver:/home/steam/valheimserver \
  -v ~/steam/Valheim:/home/steam/.config/unity3d/IronGate/Valheim \
  -p 2456:2456 -p 2456:2456/udp \
  -p 2457:2457 -p 2457:2457/udp \
  -p 2458:2458 -p 2458:2458/udp \
  -p 27015:27015 -p 27015:27015/udp \
  cm2network/steamcmd bash
  ```
* Install or update Valheim
  ```
  cd /home/steam/steamcmd
  ./steamcmd.sh +login anonymous +force_install_dir /home/steam/valheimserver +app_update 896660 validate +exit
  ```
* Start Valheim server
  ```
  cd /home/steam/valheimserver
  ./valheim_server.x86_64 -name "Server Name" -port 2456 -nographics -batchmode -world "WorldName" -password "password"
  ```
* Done!

## Other maybe useful tips

* You can detach running docker image by pressing ```ctrl+p``` and ```ctrl+q```
* You can attch detached docker images by running ```sudo docker attach steamcmd```
* If you want to set up a service script that updates the game and runs it automatically, you might want to take a look at <a target="_blank" href="https://github.com/Nimdy/Dedicated_Valheim_Server_Script">Nimdy's Dedicated_Valheim_Server_Script</a>

## Troubleshooting

* If you still can't access your server, you might need to flush or stop iptables
  ```
  # Flush iptables
  iptables -F
  ```
* No permissions to write into the mapped directories
  ```
  # You might need to chmod them
  sudo chmod -R <yourUser>:<yourUser> ~/steam/valheimserver
  sudo chmod -R <yourUser>:<yourUser> ~/steam/Valheim
  ```

## Improvements

This repo was just meant to be a small guide to start from but I would't mind improving it if needed, so all improvements are very much welcome.
