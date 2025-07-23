# Setting up Mythic C2 
 
## Intro

This documentation will go through installation and configuration part for Mythic C2.

Here, I will showcase how to download, install it, setup HTTP C2 profile for SSL communication and install the Apollo agent.


 !!!
 Important:
 Special thanks to all of my Patreon supporters.
By supporting my work on [Patreon](https://www.patreon.com/Lsecqt) you get access to internal documentation, private tools and videos.
!!!

You will need:

| Item            | Link                                     |
| --------------- | ---------------------------------------- |
| Mythic C2 Core  | https://github.com/its-a-feature/Mythic  |
| HTTP C2 Profile | https://github.com/MythicC2Profiles/http |
| Apollo Agent    | https://github.com/MythicAgents/Apollo   |

## Installation

### Installing Mythic C2 Core

Before going straight to the installation, we need to make sure we have the prerequisites. Mythic is heavily dockerised system, which simply mean that everything runs inside a docker container. In order to make it work you will need to have:

* docker
* docker-compose plugin
* gcc
* make

The following one liner should be enough:

```bash
sudo apt update && sudo apt install -y docker.io docker-compose-plugin gcc make
```

!!!
Inside the Mythic's repo there are files for automating this approach for different distros like [kali](https://github.com/its-a-feature/Mythic/blob/master/install_docker_kali.sh), always run this first especially on a brand new system!
!!!

Now we are ready to install Mythic C2:

```bash
cd /opt/
git clone https://github.com/its-a-feature/Mythic
cd Mythic
sudo make
```

If everything is alright up to this point, a `mythic-cli` binary will be generated inside your folder. This binary controls everything.

![[Pasted image 20250723202604.png]]

Then we need to install a C2 profile and an Agent.

In a nutshell, the Agent is the payload itself that will start a callback when executed on the targeted system, whereas the C2 profile is the nature of the callback, for example SMB / HTTP and e.t.c.


### Installing HTTP Profile

Open sources C2 profiles can be found here: https://github.com/MythicC2Profiles

!!!
Installation instructions for other C2 profiles are the same, however their configuration might differ
!!!

To install the HTTP profile you can follow these steps, which are also explained in its [repo](https://github.com/MythicC2Profiles/http):

```bash
cd /opt/Mythic
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http
```

![[Pasted image 20250723203212.png]]


### Installing The Apollo Agent

Open sources Agents can be found here: https://github.com/mythicagents

!!!
Installation instructions for other Agents are the same
!!!

To install the Apollo agent you can follow these steps, which are also explained in its [repo](https://github.com/MythicAgents/Apollo)

```bash
cd /opt/Mythic
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
```

![[Pasted image 20250723203442.png]]

## Configuring Mythic C2

Here I want to setup the HTTP C2 profile to accept only encrypted communication. For the demonstration purposes I will go with self signed certificate, but ideally, you would want to create a valid certificate.

To generate self signed certificate you can use this snippet:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365
```

This will output 2 file:
* cert.pem
* key.pem

Place them in `/opt/Mythic/C2_Profiles/http/c2_code/`

Then, open `/opt/Mythic/C2_Profiles/http/c2_code/config.json` with any text editor (root needed by default)

And here, make sure to have something like this:

```json
{
  "instances": [
  {
    "ServerHeaders": {
      "Server": "NetDNA-cache/2.2",
      "Cache-Control": "max-age=0, no-cache",
      "Pragma": "no-cache",
      "Connection": "keep-alive",
      "Content-Type": "application/javascript; charset=utf-8"
    },
    "port": 443,
    "key_path": "key.pem",
    "cert_path": "cert.pem",
    "debug": false,
    "use_ssl": true,
    "payloads": {}
    }
  ]
}
```

Pay attention to:

* port
* key_path
* cert_path
* use_ssl

!!!
Also, you would want to also change some of the default values, like the one for the `Server` header. Trust me.
!!!

Now you can start your whole Mythic C2, which will also fire up the HTTP C2 profile and the Apollo agent.

![[Pasted image 20250723204102.png]]

If everything is alright, you should see both dockers running, and also their status should be online from the Mythic UI

![[Pasted image 20250723204441.png]]

![[Pasted image 20250723204452.png]]


!!!
Important:
Now whenever you generate a payload, always make sure to use https prefix and the port you specified in your config file.
!!!

![[Pasted image 20250723204607.png]]


## Bonus - IPTables for Multiplayer Mode

If you want to use Mythic C2 with your team, ideally you do not want to expose the UI itself. This snippet will help:

```
iptables -I DOCKER 1 -p tcp -s "Operator IP" --dport 7443 -j ACCEPT
iptables -A DOCKER -j DROP
```

This will allow specific IP (`Operator IP`) to connect to the Mythic's UI and drop all other requests.

!!!
Dropping access to the UI do not mean that the Agent access to the C2 profile will also drop! This is another independent service!
!!!

Thank you for your time, hope this was useful.

If you appreciate my work you can [Subscribe](https://www.youtube.com/@Lsecqt) or become my [Patreon](https://www.patreon.com/c/Lsecqt)
