# Welcome to my SOAR Automation Project!


I set up a SOAR homelab referencing MyDFIR's playlist on youtube. 
This is the link to the referenced series if you would like to check it out https://www.youtube.com/watch?v=Lb_ukgtYK_U

The entire project will be documented at https://loganneallyit.net


This project all starts out with an old 2012 Mac Mini, that I wanted to convert into some sort of server. I had upgraded the ram to 16gb of DDR3 so i wanted to utilize that.

I started out with installing Ubuntu 24.04 onto a bootable drive.
After Ubuntu was all setup and updated, I got to installing TheHive. Listed below:

# Installing TheHive 5

Dependences
```bash  
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release
```
Install Java
```bash
wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
```
Install Cassandra
```bash
wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```
Install ElasticSearch
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```
***OPTIONAL ELASTICSEARCH***
Create a jvm.options file under /etc/elasticsearch/jvm.options.d and put the following configurations in that file.
```bash
-Dlog4j2.formatMsgNoLookups=true
-Xms2g
-Xmx2g
```
Install TheHive
```bash
wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive
```
Default Credentials on port 9000
credentials are 'admin@thehive.local' with a password of 'secret'

# Docker
After installing TheHive, I installed Docker and Docker-Compose
following this guide, https://documentation.wazuh.com/current/deployment-options/docker/docker-installation.html
```bash
sysctl -w vm.max_map_count=262144
uname -r
curl -sSL https://get.docker.com/ | sh
systemctl start docker
```
Now Install Docker-Compose:
```bash
curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
## Yay!! We have Docker!

Now we install Wazuh with Docker-Compose
```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.11.0
cd wazuh-docker
cd single-node (I did not do multi-node for this since this was only a demonstration)
```
(To Generate the SSL Certs)
```bash
docker-compose -f generate-indexer-certs.yml run --rm generator 
```
```bash
docker-compose up
docker-compose up -d
docker ps
```

I then configured TheHive and Wazuh to reflect the local ip address of my server.
I also changed some ports around on Wazuh to avoid conflict with cassandra, elasticsearch, and thehive

After all of that was squared away, I set up a windows 10 virtualbox.
I installed sysmon from the microsoft page: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

After configuring Wazuh and TheHive login and make sure they are both working. 
The local address of Wazuh should include https://0.0.0.0 (your configured IP address) and TheHive will be http://0.0.0.0:9000. (The same Ip Address)

If they are not working it may be due to the urls not matching up or a configuration error. Following the configurations that MyDFIR did in his video should work seamlessly.

I switched back over to my VirtualBox and set up Telemetry and Mimikatz.

Now, when creating a rule for Wazuh, you can go to /wazuh-docker/single-node/config/wazuh_cluster
The wazuh_manager.conf will be there. Do not attempt to enter the Docker Container as any changes you make will not be persistent. 

You can also use the dashboard and navigate to the server management tab on the left side of you screen and press settings.
There will be an edit configuration button on the top right side of the menu. Any edits you save in that will save. 
That's where I added the archive configuration and the shuffle webhook configuration (More on that later)
I preferred using the dashboard.

After that was all squared away, you can now move onto shuffle. Just follow MyDFIR's video.
If you are having issues with TheHive, make sure you port forward port 9000 and use your machine's PUBLIC IP not the local, ie: 192.168.1.1 it will not work.
the URl should be in the http://1.1.1.1:9000 format. The API Key is pretty straightforward. 

When setting up the TheHive, use the advanced options and the JSON format. I ran into no issues this way.



For setting up the email section, you can follow the video or use your personal. 
I used my personal and followed the same example as I did with TheHive



You can continue to follow the video, I stopped to look into Shuffle automation and what I can do with it.


# Resources:
https://wazuh.com/
https://docs.strangebee.com/thehive/download/#docker
https://www.virustotal.com/
https://shuffler.io/
https://github.com/MyDFIR/SOC-Automation-Project/tree/main

