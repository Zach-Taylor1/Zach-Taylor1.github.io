# Wireguard Server Setup
Spun up a Digital Ocean droplet with the following settings:

    Location: San Francisco
    Ubuntu 24.04 (LTS)
    Basic
    Regular: $6/Month payment option
    Normal SSD
    Password: m@Nk321y
Launched the droplet console once it was running

Ran the following commands:

    sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
.

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
.
    
    sudo add-apt-repository \
      "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
.

    apt-cache policy docker-ce
.

    sudo apt install docker-ce -y
.

    sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
.

    sudo chmod +x /usr/local/bin/docker-compose
.

    mkdir -p ~/wireguard/
.

    mkdir -p ~/wireguard/config/
.

    nano ~/wireguard/docker-compose.yml
Copy and Pasted the following information:
	
    version: '3.8'
    services:
      wireguard:
        container_name: wireguard
        image: linuxserver/wireguard
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=America/Chicago
          - SERVERURL=165.232.145.99
          - SERVERPORT=51820
          - PEERS=pc1,pc2,phone1
          - PEERDNS=auto
          - INTERNAL_SUBNET=10.0.0.0
        ports:
          - 51820:51820/udp
        volumes:
          - type: bind
            source: ./config/
            target: /config/
          - type: bind
            source: /lib/modules
            target: /lib/modules
        restart: always
        cap_add:
          - NET_ADMIN
          - SYS_MODULE
        sysctls:
          - net.ipv4.conf.all.src_valid_mark=1
Write and exit the nano

Run the following commands:

    cd ~/wireguard/
.

    docker-compose up -d
.

    docker-compose logs -f wireguard   
Downloaded the wiregaurd app on my phone

    Hit the ‘+’ button in the top left of the app
      Created a path from QR code
        Scanned the qr that was generated in the docker console
Ran the following command in my personal desktop terminal:

    scp root@165.232.145.99:/root/wiregaurd/config/wg_confs/wg0.conf /Users/zachtaylor/Documents
Downloaded wiregaurd on my laptop

    From here I had to nano the wg0.conf file and comment out the PostUp and PostDown lines in order for 
      Activated the vpn with the updated wg0.conf file
You now have a working wiregaurd vpn

