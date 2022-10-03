# rport-docker
repository to build a docker container for rport, this version contains guacd to use RDP via web browser (remember to disable nla authentication for RDP).
Fail2ban and iptables are also running to further protect rport from scanner and password guessing attacks.
You will need to add a config file.

to use this correctly you will need your own registered domain and create 2x A Records pointing to the host server IP rport.mydomain.com and connect.rport.mydomain.com the first will be used for the UI and API and second for the backend 8080. I recommend using Nginx Proxy Manager and setting up 2x reverse proxy with letsencrypt on pointing to container IP and port 3000 ( API/UI ) and to port 8080. In the advanced section you need to enable tls v1.3 for port 3000. 

docker-compose
```
version: '3.9'
services:
  rport-server:
    container_name: rport
    image: iotech17/rport:latest
    cap_add:
     - sys_nice
    ulimits:
      nproc: 65535
      nofile:
        soft: 262144
        hard: 262144
    restart: always
    privileged: true
    ports:
      - 3000:3000
      - 4822:4822
      - 10000:8080
      - 20000-20100:20000-20100
    volumes:
      - ./rportData/rportd.conf:/etc/rport/rportd.conf                                      #Store rportd.conf in docker directory
      - ./rportData/client-auth.json:/var/lib/rport/client-auth.json                        #Use encrypted password (see rport help for json format)
      - ./rportData/api-auth.json:/var/lib/rport/api-auth.json                              #Use encrypted password (see rport help for json format)
      - /home/user/nginx/letsencrypt/live/npm-7/privkey.pem:/var/lib/rport/privkey.pem      #Location of letsencrypt key for your domain
      - /home/user/nginx/letsencrypt/live/npm-7/fullchain.pem:/var/lib/rport/fullchain.pem  #Location of letsencrypt cert for your domain
      - data:/var/lib/rport/
    
    networks:
      BackendProxy:
        ipv4_address: 172.10.0.10 #DO NOT CHANGE set in rportd.conf and Nginx Reverse Proxy

volumes:
  data:

networks:
  BackendProxy: # This is the network provided by Nginx Proxy Manager (set IP to 172.10.0.0/24)
    external: true
