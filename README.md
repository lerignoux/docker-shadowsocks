### Docker image for shadowsocks

This will deploy a fully functional Shadowsocks server on a docker machine.

Author container is available on [Docker Hub](https://hub.docker.com/r/mritd/shadowsocks/)

# Server Setup


## Server setup:
- Create a [google cloud platform account](https://cloud.google.com/)
Don't forget to use a strong unique password ;)

in this project [metadata](https://console.cloud.google.com/compute/metadata/sshKeys) add your ssh private key.
I assume you know how to create a ssh private/public key, otherwise plenty of tutorials are available online.
Once you got them write down your username and paste your public key info in the project ssh keys.

I'll be using john as username for the rest of the documentation.

### Open the required ports:
In your [networking interface](https://console.cloud.google.com/networking/networks) create a project and add the following firewall rules:

shadowsocks:
- `upd:6500`
- `tcp:6443` **

For all these leave `IP ranges` filter and add `0.0.0.0/0` in `Source IP range` to allow any connection. the `<protocol>:<port>` entry must be added in
`Specified protocols and ports` section
you should already have default-allow-https and default-allow-ssh rules that are needed.

![Configuration](data/GoogleCloudFirewallConfig.png)

Ensure you write down the `targets` field for these rules to assign them to your server. Mines were
- *shadowsocks*

### Create your instance:
Create a new instance in [Compute Engine / VM Instances](https://console.cloud.google.com/compute)

Be sure to select a close by area since it will greatly affect VPN performances
Shrink down the `Machine type` according to your usage (I am using a micro instance for my VPN and it's enough IMO)

For the `Boot disk` I recommend the latest stable CoreOS since we will be using a docker image.

Once created ensure you assign the right `network tags` to it. These must be the `targets tags` previously created in the firewall configuration
for me: I added
- *shadowsocks*

Once created you should see the newly created instance in your instances list.
Write down the IP address.

** The ports are to be adapted according to the one used by the shadowsocks server (see below)

### Start your server container:
```
docker run -d --restart always --name shadowsocks -p 6443:6443 tommylau/shadowsocks  -s 0.0.0.0 -p 6443 -m aes-256-cfb -k <keytochange>
```
Replace then key <keytochange> with a secret key of your choice (and the port if needed)


# client setup
### Install shadowsocks:
```
sudo apt-get install shadowsocks
```

### Copy the template configuration
```
cp ~/Projects/docker-shadowsocks/config/shadowsocks.json.tpl ~/Projects/docker-shadowsocks/config/shadowsocks.json
```

### Update the config file content
Using your server configuration information (`password`, `port`, `address`, `local_port`)

### Run your client using:
```
sslocal -c ~/Projects/docker-shadowsocks/config/shadowsocks.json
```

you now have a local proxy on port <local_port> through your shadowsocks connection
