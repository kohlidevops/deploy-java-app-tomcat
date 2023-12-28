# To build a java app using maven and deploy a war file to Tomcat

## Launch Ubuntu22 EC2 instance

![image](https://github.com/kohlidevops/deploy-java-app-tomcat/assets/100069489/0c7057b3-0773-444e-b788-75e3565d5ddf)

## Install Java on ubuntu

SSH to machine and Install a Java11

```
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install openjdk-11-jdk -y
java -version
```

## Download Tomcat and install

To download a tomcat and install it and configure on ubuntu machine

```
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.83/bin/apache-tomcat-9.0.83.tar.gz
mkdir /etc/tomcat
sudo mkdir /etc/tomcat
sudo tar -xf apache-tomcat-9.0.83.tar.gz -C /etc/tomcat/
sudo chown ubuntu:ubuntu /etc/tomcat/ -R
sudo chmod +x /etc/tomcat/apache-tomcat-9.0.83/bin/*.sh
```

#### To find the Java home directory

```
$(dirname $(dirname $(readlink -f $(which javac))))
```

## To make the Tomcat run as a Service

'''
sudo nano /etc/systemd/system/tomcat.service
```

Copy the below contents and add it to the tomcat.service file

```
[Unit]
Description=Tomcat 9 servlet container
After=network.target

[Service]
Type=forking

User=root
Group=root

Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"

Environment="CATALINA_BASE=/etc/tomcat/apache-tomcat-9.0.83"
Environment="CATALINA_HOME=/etc/tomcat/apache-tomcat-9.0.83"
Environment="CATALINA_PID=/etc/tomcat/apache-tomcat-9.0.83/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/etc/tomcat/apache-tomcat-9.0.83/bin/startup.sh
ExecStop=/etc/tomcat/apache-tomcat-9.0.83/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```
