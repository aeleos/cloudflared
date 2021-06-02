# Instructions for Cloudflared on Unraid

docker run -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel login

docker run -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel create <name>

Returns UUID - <UUID>

nano /mnt/user/appdata/cloudflared/config.yaml

tunnel: <UUID>
credentials-file: /home/nonroot/.cloudflared/<UUID>.json

ingress:
  - service: https://192.168.1.101:443
    originRequest:
      noTLSVerify: true


CA Apps
Search cloudflared
Search  Docker hub
Import cloudflare/cloudflared

Add path: 
Host:  /mnt/user/appdata/cloudflared
Container: /home/nonroot/.cloudflared/

Advanced:
Post arguments: tunnel run <UUID>

DNS w/ Cloudflare
CNAME points to <UUID>.cfargotunnel.com

