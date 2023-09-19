# Duckiebot
Two years after completing the [Self-Driving Cars with Duckietown MOOC on edx](https://www.edx.org/learn/technology/eth-zurich-self-driving-cars-with-duckietown) I decided to dust off my Duckiebot and go back to play with it to apply some of the things I learned in the last couple of years.

This repository contains the notes I collected to document this endeavor. 

## Which version?

It seems that the version I own is the [DB21M](https://docs.duckietown.com/daffy/opmanual-duckiebot/preliminaries_hardware/duckiebot_configurations/index.html#duckiebot-config-db21m).

## Power up

The first challenge was charging the battery after two years in order to power up the bot.

I plugged the bot and tried to power it up - Note: using the side button in the battery, not the button at the top which is only to power down. 

![](./assets/power_up.jpg)

The robot wouldn't boot. The LEDs lit up in odd colours, but the screen didn't work. After a while I noticed the fan in the Jetson would start to power up briefly and then power down again.

[Handling instructions in the docs](https://docs.duckietown.com/daffy/opmanual-duckiebot/operations/handling/db21.html#handling-duckiebot-db21) were not fully straight forward. After searching the docs and Slack, I came to the conclusion that the battery was totally drained and followed the advice in [the docs FAQ](https://docs.duckietown.com/daffy/opmanual-duckiebot/debugging_and_troubleshooting/faq/index.html) and in [Slack](https://stackoverflowteams.com/c/duckietown/a/230/1434) to unplug the HUT cables going to motors and Jetson, and letting only the cables needed to charge the battery for 5+ hours.

![](./assets/charging_battery.jpg)

After about 30' charging in this way, I plugged the cables and tried again to power up. This time the fan started, the wifi dongle started to blink blue... and then it shut off - before the screen ever switching on. Well, it seems I am moving in the right direction, but we need the battery to charge longer.

Note: browsing the [section on circuits and batteries](https://docs.duckietown.com/daffy/opmanual-duckiebot/preliminaries_hardware/circuits_batteries/index.html) of the docs I came across the description of the so-called *battery protection mode* by which a depleted battery may go into hibernation and remain unresponsive during about 30' while trickle charging to a safe level.

Note: USB cables from left to right 

1. to Jetson
2. to battery
3. to mains
4. to motors and LEDs

Unplug 3 from the mains when the battery is charged then connect cables in this order: 2 - 4 -1 (cfr steps 65-67 of the [assembly instructions](https://docs.duckietown.com/daffy/opmanual-duckiebot/assembly/db21m/index.html#howto-power-db21m))

To be continued...

After the whole night charging, I tried again but the booting process did not complete. There was no blinking of the wifi dongle and when I plugged in a screen to see what was going on the screen froze with a Nvidia logo.

I assumed at some point the SD card got corrupted - perhaps one of the times when I forcefully unplugged the Jetson...

## Software

I decided to install software in the laptop and flash the SD again following [the docs](https://docs.duckietown.com/daffy/opmanual-duckiebot/setup/setup_laptop/index.html) step by step.

Note: I am using Ubuntu 20.04 instead of the recommended 22.04.

### Install dependencies 

```bash
$ sudo apt update
$ sudo upgrade
$ sudo apt install -y python3-pip git git-lfs curl wget
$ pip3 version
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
```

### Install Docker

Remove any older versions of Docker (there were none)

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

Set up the apt repo containing Docker

```bash
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```

Add the official GPG key

```bash
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Set up the repository

```bash
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update again and install Docker Engine and Docker Compose:

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ sudo apt-get install docker-compose
```

Add the current user to the `docker` user group

```
sudo adduser `whoami` docker
```

Important: **restart** for the change to take effect

```bash
$ docker --version
Docker version 24.0.6, build ed223bc
$ docker buildx version
github.com/docker/buildx v0.11.2 9872040
```

It is ok if docker version is > v1.4.0 and `buildx` version > v.0.8.0

Test docker:

```bash
$ docker run hello-world

Hello from Docker!
...
```

### Install Duckietown shell

```bash
$ pip3 install --no-cache-dir --user --upgrade duckietown-shell
```

Do `$ nano ~/.bashrc` to add the following lines to the end of `~/.bashrc` in order to make local binaries accessible to the system:

```
...
# make Duckietown shell local binaries accesible
export PATH=~/.local/bin:${PATH}
```

Save the file and source it by closing and reopening the terminal or running `$ source ~/.bashrc` to apply the change.

Check it works: 

``````bash
$ which dts
/home/mhered/.local/bin/dts
``````

Set version to `daffy` :

```bash
$ dts --set-version daffy
Duckietown Shell (v5.5.10)
INFO:dts:Configured dts to version: daffy
```

Log in to [Duckietown](https://hub.duckietown.com/profile/) with the Github user, copy the Duckitown token and pass it to `dts` with:

```bash
$ dts tok set
...
Enter token: ****************

dts :  Correctly identified as uid = ****

```

Checkpoint:

```bash
$ dts version
Duckietown Shell (v5.5.10)
INFO:dts:Commands version: daffy
INFO:dts:Checking for updates in the Duckietown shell commands repo...
INFO:dts:Duckietown shell commands are up-to-date.
...
```

```bash
$ dts tok status
...
dts :  Correctly identified as uid = 3372
```

### Configure DockerHub

Login to [DockerHub](https://hub.docker.com/)  (login:mhered/pwd: >9 letters)

Create an Access Token : username -> Account Settings -> Security -> New Access Token

Test the token from the docker CLI:

```bash
$ docker login -u mhered
Password: 
...
Login Succeeded
```

Provide the docker credentials (login and token) to `dts`: 

```bash
$ dts config docker credentials set \
    --username mhered \
    --password ****************    
...
INFO:dts:Docker access credentials stored!
```

Checkpoint:

```bash
$ dts config docker credentials info
...
INFO:dts:Docker credentials:

	registry:   docker.io
	username:   mhered
	  secret:   ****************
```

Note: Duckietown and dts credentials are stored unencrypted in `~/.dt-shell/config.yaml`

Note: repeat this process if a new access token needs to be generated. 

### Flash SD card

The GUI failed to launch with a page not found error (why?):

```bash
$ dts init_sd_card --gui
...
 File "/home/mhered/.local/lib/python3.8/site-packages/dockertown/utils.py", line 220, in run
    :      raise DockerException(
    :  dockertown.exceptions.DockerException: The docker command executed was `/usr/bin/docker exec c8ce09d93cf92fa64615eac9417fe079329dfe30de65c8eebfbe511c41581864 sleep infinity`.
    :  It returned with code 1
    :  The content of stdout is ''
    :  The content of stderr is 'Error response from daemon: Container c8ce09d93cf92fa64615eac9417fe079329dfe30de65c8eebfbe511c41581864 is not running
    :  '
WARNING:dts:Please update "pip" to have better debug info.

dts :  To report a bug, please also include the contents of /home/mhered/shell-debug-info.txt
```

So I used the CLI:

```bash
$ dts init_sd_card --hostname patobot --type duckiebot --configuration DB21M --wifi HUAWEI:***********h8 --country ES
```

Accept 3rd party licenses, indicate SD size (32) and select the proper device.

The process involves 3 steps: download, flash and verify and took 2h? (started at 21:30)

Default username and password are `duckie` and `quackquack`
