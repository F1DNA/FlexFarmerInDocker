# FlexFarmerInDocker
How to Setup Flexfarmer in Docker

Written using Ubuntu Server – Ubuntu Desktop should be identical, for Windows you will need to install docker first which I won't be covering here, then you should be able to skip my install docker section and follow the rest of this guide with minor changes as needed. I.E., file paths and such will be different. 

*Ubuntu server for Arm / Raspbian can use this guide as well*

In addition to installing docker, we are going to setup 3 containers in total:

**Portainer** 

•	For those experienced with Docker, this is not necessary nor should you need my guide at all!  But for those new to Docker, this will make it very easy to use as it provides a GUI via web browser.  We will be utilizing Portainer in a Docker-Compose fashion.  Docker-Compose is way to setup containers via a configuration file instead of a long command

**Watchtower**

•	Watchtower will check your containers for updates on a schedule, download the new images and restart the container with the new image.  This makes your setup hands off.  When a new Docker image is available, it will be downloaded and then your container is restarted.  In my farm, this restart takes approx 10 seconds before flexfarmer is back up and functioning on the blockchain.  Larger farms, more drives, etc may take a bit longer but this is still very quick.

**FlexFarmer**

•	This is why you are here, right?  

**Install Docker**

Reference the official docker documentation for more details if you would like, or continue on as I have copied the necessary steps from the official guide here

https://docs.docker.com/engine/install/ubuntu/

Setup the repository

1.	Update the apt package index and install packages to allow apt to use a repository over HTTPS
```
sudo apt-get update
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg \
lsb-release
```

2.	Add Docker’s official GPG Key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3.	Use the following command to set up the stable repository, make sure you grab the correct one for your CPU architecture

a.	X86_64 / amd64 

```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
b.	Arm64 (Rasp Pi and the like)
```
echo \
  "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker Engine

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

That’s it, Docker is installed!

You can verify this simply by typing `docker` which will show the basic docker help file

**Install Portainer**

Check out the readme on GitHub or continue on as I have copied the necessary steps from the official guide

https://github.com/portainer/portainer/blob/develop/README.md

1.	Create the portainer volume

```
sudo docker volume create portainer_data
```

2.	Create the container

```
sudo docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

Portainer is now installed.  To access portainer, open a browser on any computer on your network and browse to your FlexFarmer machine’s IP or hostname, specifying the port 9000.

Examples: 

http://192.168.11.63:9000

http://Myflexfarmermachine:9000

Create username and password

 ![image](https://user-images.githubusercontent.com/61926834/132616293-0f5caa80-7bf4-4e7d-b03e-23e88d727546.png)


Select Docker:

 ![image](https://user-images.githubusercontent.com/61926834/132616303-a22b32b5-7ce6-49a2-8a38-da83f5586509.png)


Click anywhere on your local docker instance to open it up:

 ![image](https://user-images.githubusercontent.com/61926834/132616319-759dba5d-4465-4d26-a9ce-e56e8d41f2bf.png)


Click on stacks.  Stacks is Portainer’s version of Docker-Compose:

 ![image](https://user-images.githubusercontent.com/61926834/132616356-2b68f051-8935-4382-9790-e6ab94013ec9.png)

Add stack:

 ![image](https://user-images.githubusercontent.com/61926834/132616376-6d6a58d3-09be-4e82-b18a-4cc7e672fd3d.png)


Name it “watchtower” (DO NOT USE ANY CAPITAL LETTERS)

Paste the following into web editor.  The only thing you should need to change is the Timezone line to wherever you are.  If you take out the environment and -TZ lines, it will use UTC which is also fine.  As well, you can change the –interval to however many seconds you want.  I have this setup for 21,600 seconds or 6 hours.

```
version: '3'
services:
   watchtower:
      container_name: watchtower
      restart: always
      image: containrrr/watchtower
      environment:
      - TZ=America/Chicago # Change this to your timezone as needed
              # For a list of timezone options - http://manpages.ubuntu.com/manpages/hirsute/man3/DateTime::TimeZone::Catalog.3pm.html
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=21600 #This value controls how often in seconds Watchtower checks for new images - Currently set to 6 hours 
      - WATCHTOWER_DEBUG=true
      volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```


Note that spacing matters as this is YAML code:

![image](https://user-images.githubusercontent.com/61926834/132967848-8eca98b7-360a-4406-8c0d-80996e4c558f.png)


Scroll down and click “Deploy the stack”

 ![image](https://user-images.githubusercontent.com/61926834/132616406-be4fa1fd-48b4-4ffc-b8fa-b3934961a85f.png)


That’s it, Watchtower will now check for new images for all 3 of your containers every 6 hours. Navigate to “Containers” on the left column to see your running containers:
 
![image](https://user-images.githubusercontent.com/61926834/132616412-fbde82bd-7326-4d16-a663-f41fd8f04771.png)

Create FlexFarmer Container:

Now we have a bit of setup to do. 

1.	Navigate to your home folder

```
cd /home/username
```

2.	Create a directory called flexfarmer and CD into it

```
mkdir flexfarmer
cd flexfarmer
```

3.	Create your config file:

```
nano config.yml
```

You will place your config.yml in here.  However you go about creating the config.yml is up to you.  I will walk you through the easiest method which is to use the flexpool.io tool.

4.	Navigate to https://www.flexpool.io

a.	Click on Start Faming in the Chia section

b.	Click on the FlexFarmer option

c.	Leave OS on Linux

d.	Skip Downloading FlexFarmer as we will be pulling the image from dockerhub instead of manually downloading and installing

e.	Scroll down and paste your mnemonic phrase in (Feel free to download the tool to your machine instead of using the browser for a more secure process)

f.	Then follow the rest of the tool to create your base config file template

g.	Obviously step 10 on the webpage we will not be doing as we have not installed flexfarmer yet and this will be run for you automagically when the container starts

5.	Back in nano, copy/paste the config file info from the webpage into nano

7.	Bonus: Enable log files, highly recommend doing this.  Just add this line to the end of your config file:

```
log_file_path: /flexfarmer.log # Write logs to a specific file
```

7.	Press “ctrl+x” to exit nano and then press “y” to save

8.	Create the initial flexfarmer.log file:

```
touch flexfarmer.log
```

Back in Portainer, let’s get the Flexfarmer container setup:

1.	Navigate back to stacks and “Add Stack”
2.	Name it “flexfarmer” – Remember, no caps
3.	Paste in the following:

```
version: "3"

services:
  flexfarmer:
    image: flexpool/flexfarmer:latest
    container_name: flexfarmer
    volumes:
      - /home/f1dna/flexfarmer/config.yml:/config.yml #Change f1dna to your username
      - /home/f1dna/flexfarmer/flexfarmer.log:/flexfarmer.log #change f1dna to your username - delete or comment out entirely if not enabling in config.yml
      - /mnt:/mnt
    command:
      -c /config.yml
    environment:
      - TZ=America/Chicago #Change this to your timezone as needed
        # For a list of timezone options - http://manpages.ubuntu.com/manpages/hirsute/man3/DateTime::TimeZone::Catalog.3pm.html
    restart: unless-stopped
```

So again, we have some options here.  First, I mapped all of /mnt to the container.  You COULD specify each mount point.  I.E.:

-	/mnt/1stfolder:/mnt/1stfolder
-	/mnt/2ndfolder:/mnt/2ndfolder
-	/mnt/etc:/mnt/etc

Alternatively, doing just /mnt makes it easier if you add mount points (additional drives) to your setup later on, you only need to edit the config.yml file and restart the container.  Otherwise you also have to update the stack config.

And of course, TZ=America/Chicago – you need your region/city or just remove both the environment and the TZ line to use UTC.  Note that this will effect the time stamps in the flexfarmer.log so perhaps take a moment to set this correctly unless your brain thinks in UTC easily.  I can go either way.

4.	When you are ready, hit update stack
5.	Back in the terminal from your flexfarmer folder, run:

```
tail -f -n +0 flexfarmer.log
```
6.	Make sure everything looks good here.  No invalid plots, all plot folders showing up, total space looks right, signage points signing, partials submitting, etc
7.	If you decide not to enable logging and eliminate the flexfarmer.log, you can also check the container log in portainer.  I'm not sure the limitations of that log, how many lines it retains, etc.  I would really encourage enabling the log file in your config incase you need to go back in time to investigate something.



**Should you feel compelled, you may send a tip in the form of XCH to:**

xch1u2jhwq9q37v822ejvjytu289th4wad8azuyu7m0gmjuzzzd76dkqx06phs
