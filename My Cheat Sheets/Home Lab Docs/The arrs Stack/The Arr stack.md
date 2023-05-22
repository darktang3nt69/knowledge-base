```yaml
---

version: "3.9"
networks:
  default:
    name: the-arrs
    external: true
services:
#---------------------------------------------------------------------#
#     Sonarr - TV shows manager                                       #
#---------------------------------------------------------------------#
  sonarr:
    container_name: sonarr
    image: cr.hotio.dev/hotio/sonarr:latest
    restart: unless-stopped
    logging:
      driver: json-file
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
      - UMASK=022
      - DOCKER_MODS=gilbn/theme.park:sonarr
            - TP_DOMAIN=gilbn.github.io
            - TP_THEME=aquamarine
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/lokesh/sambashare/the-arrs-config/sonarr-config:/config
      - /home/lokesh/sambashare/data:/data
    ports:
      - '8989:8989'
#---------------------------------------------------------------------#
#     Radarr - Movies manager                                         #
#---------------------------------------------------------------------#
  radarr:
    container_name: radarr
    image: cr.hotio.dev/hotio/radarr:latest
    restart: unless-stopped
    logging:
      driver: json-file
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
      - UMASK=022
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/lokesh/sambashare/the-arrs-config/radarr-config:/config
      - /home/lokesh/sambashare/data:/data
    ports:
      - '7878:7878'
      
#---------------------------------------------------------------------#
#     Bazarr - Subs Manager for media library                         #
#---------------------------------------------------------------------#
  bazarr:
    container_name: bazarr
    image: cr.hotio.dev/hotio/bazarr:latest
    restart: unless-stopped
    logging:
      driver: json-file
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/lokesh/sambashare/the-arrs-config/bazarr-config:/config
      - /home/lokesh/sambashare/data/media:/data/media
    ports:
      - '6767:6767'
    
#---------------------------------------------------------------------#
#     Overseerr - is a request management and media discovery tool    #
#                 built to work with your existing Plex ecosystem     #
#---------------------------------------------------------------------#
  overseerr:
    container_name: overseerr
    image: cr.hotio.dev/hotio/overseerr
    restart: unless-stopped
    ports:
      - "5055:5055"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Asia/Kolkata
    volumes:
      - /home/lokesh/sambashare/the-arrs-config/overseerr-config:/config

#---------------------------------------------------------------------#
#     Requestrr - is a chatbot used to simplify using services like   #
#                 Sonarr/Radarr/Overseerr/Ombi via the use of chat!   #
#---------------------------------------------------------------------#

  requestrr:
    image: lscr.io/linuxserver/requestrr:latest
    container_name: requestrr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /home/lokesh/sambashare/the-arrs-config/requesterr-config:/config
    ports:
      - 4545:4545
    restart: unless-stopped

#---------------------------------------------------------------------#
#    Prowlarr -is an indexer manager/proxy built on the popular *arr  #
#    .net/reactjs base stack to integrate with your various PVR apps  #
#---------------------------------------------------------------------#
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /home/lokesh/sambashare/the-arrs-config/prowlerr-config:/config
    ports:
      - 9696:9696
    restart: unless-stopped

#---------------------------------------------------------------------#
#     Homarr - A simple, yet powerful dashboard for your server.      #
#---------------------------------------------------------------------#
  homarr:
    container_name: homarr
    image: ghcr.io/ajnart/homarr:latest
    restart: unless-stopped
    volumes:
      - /home/lokesh/sambashare/the-arrs-config/homarr-config:/app/data/configs
      - /home/lokesh/sambashare/the-arrs-config/homarr-icons:/app/public/icons
    ports:
      - '7575:7575'

#---------------------------------------------------------------------#
#     qbittorrent - Plain old torrent downloader.                     #
#---------------------------------------------------------------------#
  qbittorrent:
    container_name: qbittorrent
    image: cr.hotio.dev/hotio/qbittorrent
    restart: unless-stopped
    ports:
      - '32773:8080'
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Asia/Kolkata
    volumes:
      - /home/lokesh/sambashare/the-arrs-config/qbitty-config:/config
      - /home/lokesh/sambashare/data/torrents:/data/torrents/

#---------------------------------------------------------------------#
#     FlareSolverr is a proxy server to bypass Cloudflare and         #
#                          DDoS-GUARD protection.                     #
#---------------------------------------------------------------------#

  flaresolverr:
      container_name: flaresolverr
      ports:
          - '8191:8191'
      environment:
          - LOG_LEVEL=info
      restart: unless-stopped
      image: 'ghcr.io/flaresolverr/flaresolverr:latest'
```