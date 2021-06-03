# Instructions for Cloudflare Tunnels (formerly known as Argo Tunnels) on Unraid

Argo Tunnel creates a secure, outbound-only connection between your services and Cloudflare by deploying a lightweight connector in your environment. With this model, your team does not need to go through the hassle of poking holes in your firewall or validating that traffic originated from Cloudflare IPs.

# Set-Up Steps

1. In unraid temrinal, run the following command to authorise the container and pull the cert

```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel login 
```
It will print out a link, this will take you to cloudflare to authorise your container. Once you choose your domain, it will send the cert to the container (in the persistent volume we set up).

2. Now we need to create a tunnel. To do this we will run another command from the unraid terminal

```
docker run -it --rm -v /mnt/user/appdata/cloudflared:/home/nonroot/.cloudflared/ cloudflare/cloudflared tunnel create <name>
```

This will return your tunnels UUID. Keep the UUID on hand as we will need it to refer back to this tunnel in later steps.

3. Now we need to create a config.yaml to configure the tunnel

```
nano /mnt/user/appdata/cloudflared/config.yaml
```
4. Now paste in the following and amend your reverse proxy IP:PORT and tunnel UUID

```
tunnel: UUID
credentials-file: /home/nonroot/.cloudflared/UUID.json

ingress:
  - service: https://REVERSEPROXYIP:PORT/
    originRequest:
      noTLSVerify: true
```
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

If you have an A record already, you can remove this as it is now not needed.

Replace your A record with a CNAME record, that points to the domain root (@) and for the content, you need to add UUID.cfargotunnel.com (inserting your UUID that was copied earlier).
  
## Example
  
|Type|Name|Value|TTL|Status|
| --- | --- | --- | --- | --- |
|CNAME|@|UUID.cfargotunnel.com|Automatic|Orange ☁️|
|CNAME|plex|@|Automatic|Orange ☁️|
|CNAME|portainer|@|Automatic|Orange ☁️|
|CNAME|radarr|@|Automatic|Orange ☁️|
|CNAME|sonarr|@|Automatic|Orange ☁️|
  
You should now be able to access all of your apps without needed a port forward!
