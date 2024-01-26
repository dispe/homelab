# homelab

## Setup:
- install docker
- install portainer agent
- configure portainer GUI extension from local machine's docker hub to connect to the remote portainer agent
- install Nginx Proxy Manager
- setup DNS and SSL

<div style="background-color: #ffffcc; padding: 10px;">
:warning: **WARNING: Do not install docker through snap. Install from docker site.
</div>

### Install Portainer Agent

https://docs.portainer.io/admin/environments/add/docker/agent

This method is used in order to manage multiple docker contexts.
The Portainer Core can be anywhere and add to it more environments. For example could be on Docker Hub application on local machine installed as extension.

In Portainer go to Settings > Environments > Add Environment 
- Select Docker Standalone
- Press Start Wizard
- Copy command, should looks like:

```bash
$ docker run -d \
  -p 9001:9001 \
  --name portainer_agent \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:2.19.4
```
- Execute on the ubuntu-docker server
  
Comeback to Portainer GUI and continue with:
- Fill in Name: ubuntu-docker
- Environment address: 192.168.1.198:9001
- Connect

### Install Ngnix Proxy Manager

https://nginxproxymanager.com/setup/


```bash
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    # Uncomment the next line if you uncomment anything in the section
    # environment:
      # Uncomment this if you want to change the location of
      # the SQLite DB file within the container
      # DB_SQLITE_FILE: "/data/database.sqlite"

      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

### Setup DNS and SSL

Go to https://www.duckdns.org/domains and register your domain

Go to Ngnix Proxy Manager - 192.168.1.198:81

SSL Certificates > Add SSL Certificate > Let's Encrypt

- Domain Names: your_domain.duckdns.org, *.your_domain.duckdns.org
- Email address for Let's Encrypt
- Use a DNS Challenge = true
- DNS Provide = DuckDNS
- Propagation Seconds = 120
- Agree and Save

:warning: If install PiHole for example, you should config proxy pass:

```json
location / {
    proxy_pass http://PIHOLE_IP:WEB_PORT/admin/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_hide_header X-Frame-Options;
    proxy_set_header X-Frame-Options "SAMEORIGIN";
    proxy_read_timeout 90;
}
```
