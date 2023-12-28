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

```
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

### To start the Tomcat service

To restart the daemon to notify systemd that a new unit file exists and start the Tomcat service

```
sudo systemctl daemon-reload
sudo systemctl start tomcat.service
sudo systemctl status tomcat.service
```

### To access the Tomcat webserver

If its running properly, then you can access the webserver using VM IP with port 8080

![image](https://github.com/kohlidevops/deploy-java-app-tomcat/assets/100069489/972de574-7807-4993-8998-a7504b5539c4)

## To create a Java Application

### Install a Java build tool - Maven

I will use the Java build tool as Maven. So I have to install the maven on ubuntu machine

```
sudo apt-get install maven
mvn -version
```
### To generate a project

Maven has a feature that can able to generate an initial maven project with a folder structured. You can use following line

```
mvn archetype:generate -DgroupId=com.app.example -DartifactId=java-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

If you execute tree command then it will show a folder structure.

![image](https://github.com/kohlidevops/deploy-java-app-tomcat/assets/100069489/309b5e2e-9a10-4b03-b429-300d66632689)

### To configure the Application

POM.XML file is the fundamental unit of work in Maven. It is an XML file that contains information about the project and configuration details used by Maven to build the project.

```
<?xml version = "1.0" encoding = "UTF-8"?>
<project xmlns = "http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"

xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

   <groupId>com.tutorialspoint</groupId>
   <artifactId>hello-world</artifactId>
   <version>1</version>
   <packaging>war</packaging>
   
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.3.0.RELEASE</version>
      <relativePath/> 
   </parent>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <java.version>1.8</java.version>
      <tomcat.version>9.0.37</tomcat.version>
   </properties>

   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>  
         <groupId>org.springframework.boot</groupId>  
	 <artifactId>spring-boot-starter-tomcat</artifactId>  
	 <scope>provided</scope>  
      </dependency>   
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>
   </dependencies>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>
   
</project>
```

### To create a application

```
sudo nano src/main/java/com/app/example/App.java
```

To use the below code to change the default java application

```
package com.app.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class App extends SpringBootServletInitializer {
   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
      return application.sources(App.class);
   }
   public static void main(String[] args) {
      SpringApplication.run(App.class, args);
   }

   @RequestMapping(value = "/")
   public String hello() {
      return "<center>Hello Latchu</center>";
   }
}
```

### To build the application using maven tool

To build the application using below command that will download application dependencies, compile, test, and package(.war file) it as a distributed application.

```
cd java-app
mvn package
```

![image](https://github.com/kohlidevops/deploy-java-app-tomcat/assets/100069489/8e8659f4-8f16-4ddf-b088-6774f18ecb2c)

## To deploy the application

The result of the previous step is a distributed application with .war extension stored in the target folder under java-app. That .war application will later be deployed on the Tomcat service.

```
cd java-app
sudo cp target/hello-world-1.war /etc/tomcat/apache-tomcat-9.0.83/webapps/
```

### To restart the Tomcat service

```
sudo systemctl restart tomcat
```

### To access your application

Now you can access your application through tomcat webserver

![image](https://github.com/kohlidevops/deploy-java-app-tomcat/assets/100069489/3dc02d11-163b-4540-93fe-f9264108bffd)

That's it!

