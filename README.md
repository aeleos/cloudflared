# Instructions for Cloudflare Tunnels (formerly known as Argo Tunnels) on Unraid (Legacy Tunnels

Argo Tunnel creates a secure, outbound-only connection between your services and Cloudflare by deploying a lightweight connector in your environment. With this model, your team does not need to go through the hassle of poking holes in your firewall or validating that traffic originated from Cloudflare IPs.

### Support Me

If you liked this guide and want to say thanks feel free to buy me a coffee https://www.buymeacoffee.com/aeleos

## Note
This tutorial is only for legacy tunnels. For the current tunnels, all you need to do is go through the official CF guide and modify the template run command such that you start the container with your key rather than a configuration file. Then you can create an tunnels and routes through the CF panel. 

Example command:
```
tunnel run --token ${CF_TUNNEL_TOKEN}
```

# Video Guide

If you would like to watch a video detailing all this information and more, check out IBRACORP's guide https://www.youtube.com/watch?v=RQ-6dActAr8

# Prerequisite

For this setup, you need to have a domain which is managed by CloudFlare, and can be done on the free plan. 

# Set-Up Steps

1. In Unraid terminal, run the following command to authorise cloudflared with the cloudflare site you want to setup with a tunnel.

```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared:2021.6.0 tunnel login 
```
It will print out a link to cloudflare. Put this link in your web browser, and select which domain you want to use. Then, the daemon will automatically pull the certificate.

2. Now we need to create a tunnel. To do this we will run another command from the unraid terminal

```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared:2021.6.0 tunnel create TUNNELNAME
```

This will create your tunnels UUID.json file, which contains a secret used to authenticate your tunneled connection with cloudflare. The JSON file is only needed for running the tunnel, but any tunnel modifications require the cert.pem. More information about what requires what can be found [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/create-tunnel).

Make sure you copy your UUID, as this will be used in later steps. It can always be found later by the name of the JSON file.

3. Now we need to create a config.yaml to configure the tunnel

```
nano /mnt/user/appdata/cloudflared/config.yaml
```
4. Now paste in the following and amend your reverse proxy IP:PORT, tunnel UUID and domain name if applicable

   - if you have an ssl certificate on your reverse proxy, you need to pass in your domain name that the SSL cert is under
   - if you want to proxy to an http server, use the commended ingress rule
   - if you want to disable ssl verification, add noTLSVerify under originRequest

```
tunnel: UUID
credentials-file: /home/nonroot/.cloudflared/UUID.json

# NOTE: You should only have one ingress tag, so if you uncomment one block comment the others

# forward all traffic to Reverse Proxy w/ SSL
ingress:
  - service: https://REVERSEPROXYIP:PORT
    originRequest:
      originServerName: yourdomain.com
      
#forward all traffic to Reverse Proxy w/ SSL and no TLS Verify
#ingress:
#  - service: https://REVERSEPROXYIP:PORT
#    originRequest:
#      noTLSVerify: true

# forward all traffic to reverse proxy over http
#ingress:
#  - service: http://REVERSEPROXYIP:PORT

 ```
 See [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/ingress) for more information about ingress rules and how they can be configured
 
 
5. Now, we need to install the app inside the Unraid UI.

   - Go to the CA Apps Tab

   - Search for cloudflared

   - Install from aeleos' Repository

   - Make the following changes:
 
6. Now we need to change the "Post Arguments". To do this we need to enable the "Advanced View" in the top right corner.

   - You should see the below command inside of "Post Arguments". Replace UUID the the UUID for your tunnel generated in step 2.

```
Post arguments: 
tunnel run UUID
```

7. Now you can start your container and if all done correctly with no errors, you should have a running tunnel!

8. The next step will be to edit your domain DNS records.

   - If you have an A record already, you can remove this as it is now not needed.

   - Replace your A record with a CNAME record, that points to the domain root (@) and for the content, you need to add UUID.cfargotunnel.com (inserting your UUID that was copied earlier).
  
## Example
  
|Type|Name|Value|TTL|Status|
| --- | --- | --- | --- | --- |
|CNAME|@|UUID.cfargotunnel.com|Automatic|Orange ☁️|
|CNAME|plex|@|Automatic|Orange ☁️|
|CNAME|portainer|@|Automatic|Orange ☁️|
|CNAME|radarr|@|Automatic|Orange ☁️|
|CNAME|sonarr|@|Automatic|Orange ☁️|
  
You should now be able to access all of your apps without needed a port forward!

# Troubleshooting

## Certificate not valid for any names

If you see an error like 

```
in the logs of cloudflared i have this error "ERR  error="Unable to reach the origin service. The service may be down or it may not be responding to traffic from cloudflared: x509: certificate is not valid for any names, but wanted to match youdomain.com
```

Try changing yourdomain.com to host.yourdomain.com, where host is a valid subdomain that you have a DNS record for. Despite this being a specific hostname, cloudflared should be able this name to verify certificates for your other subdomains as they pass through the tunnel.

Another user has reported that they fixed this issue when using NPM by enalbing HTTP/2 and installing a CF origin certificate into their NPM instance. You may also get this error for other misconfigurations in the certificate returned by NPM.

## Tunnel Create does not return an ID

If when running the commands in your terminal, you aren't returned an ID, try running the command with the container version specified, like this

```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared:2021.6.0 tunnel
```

# Enabling SSH Access via Web Rendered Terminal

1. Create a DNS record for the subdomain you want to go to for SSH access. Below is an example

|Type|Name|Value|TTL|Status|
| --- | --- | --- | --- | --- |
|CNAME|ssh|domain.com|Automatic|Orange ☁️|

2. Add Ingress Rule

Ingress rules resolve top down, so this rule should be above the - service: https://REVERSEPROXYIP:PORT/ rule.

```
  - hostname: ssh.domain.com
    service: ssh://SSHIP:PORT

```

3. Sign up for Cloudflare Teams

   - Can be done [here](https://www.cloudflare.com/teams-pricing/), free plan works up to 50 users but needs billing details, you may be able to get it to work with no plan

4. Add an Teams application for your SSH service and enable browser based rendering

   - Guide by cloudflare can be found [here](https://developers.cloudflare.com/cloudflare-one/tutorials/ssh)

# List and delete tunnels
To **list** all configured tunnels and see active connections:
```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel list
```

ID|NAME|CREATED|CONNECTIONS
---|---|---|---
NAMEID in hex|NAME of the tunnel|TIMESTAMP Date and time created|NUMBERxCFPOINT

To **revoke and delete** a tunnel:
```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel delete TUNNELID
```

If there are still active connections on the tunnel you need to force the deletion. Connections will be dropped:
```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel delete -f TUNNELID
```

**Deleting the Tunnel also invalidates the credentials file associated with that Tunnel, meaning those connections can not be re-established.**
# Credit

 - Hawks for helping beta test the setup and documentation contributions
 - DiscDuck for documentation contributions
