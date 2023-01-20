To start this handson lab,you need following resources.

- 1. Google cloud (or any other cloud or some VM's from your local machine).
- 2. Jenkins VM should have 2 CPU / 4 GB memory / 40 gb Disk. Install Jenkins in **rocky linux 8 and above** OS machine.
- 3. Class 6 and Class 7 must be completed and ready
- 4. Make sure you have root access or root login credentials.
- 5. For this Tomcat and Artifactory VM you must have 2 CPU and minimum 7.5 GB memory.Create this VM with **rocky linux 8 and above** OS machine

*******************************************************************************************************************
- Step 1 : Install pre-requisite software packages
- Step 2 : Configure environment variables
- Step 3: Configure Artifactory and TOMCAT Software Repository
- Step 4: Install Artifcatory
- Step 5: Change Java Heap Size
- Step 6: Install TOMCAT Web Application server
- Step 7: Restart Tomcat and Artifactory Services
- Step 8: Create new pipeline and enable manual deployment to deploy application in tomcat web application server
- Step 9: Create new pipeline and enable auto deployment to deploy application in tomcat web application server


*******************************************************************************************************************

Artifactory

## Step 1 : Install pre-requisite software packages
```
yum install java-11-openjdk-devel vim wget git -y
systemctl stop firewalld;systemctl disable firewalld
wget https://archive.apache.org/dist/maven/maven-3/3.8.7/binaries/apache-maven-3.8.7-bin.tar.gz
tar xvf apache-maven-3.8.7-bin.tar.gz
mkdir -p /usr/local/apache-maven
cd apache-maven-3.8.7/
mv *  /usr/local/apache-maven
```
*******************************************************************************************************************

## Step 2 : Configure environment variables
```
vim ~/.bash_profile
```
In this file
```
export JAVA_HOME=/usr/local/jdk-11.0.1/
export JRE_HOME=/usr/lib/jvm/jre-11/
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$JRE_HOME/bin
export M2_HOME=/usr/local/apache-maven
export M2=$M2_HOME/bin
export PATH=$M2:$PATH
```
```
source ~/.bash_profile
```
```
mvn -version
```
*******************************************************************************************************************
## Step 3: Configure Artifactory and TOMCAT Software Repository

just refernece - https://jfrog.com/community/download-artifactory-oss/

```
wget https://releases.jfrog.io/artifactory/artifactory-rpms/artifactory-rpms.repo 
mv artifactory-rpms.repo  /etc/yum.repos.d/
```

Note ::::::
{1)After giving wget https://releases.jfrog.io/artifactory/artifactory-rpms/artifactory-rpms.repo -O jfrog-artifactory-rpms.repo you are able to see Saving to: ‘jfrog-artifactory-rpms.repo’
2) cat /etc/yum.repos.d/jfrog-artifactory-rpms.repo (take base url and check in web browser you can seel all repo)
3)now you aee able to see jfrog artifactory }




*******************************************************************************************************************

## Step 4: Install Artifcatory
```
sudo yum update && sudo yum install jfrog-artifactory-oss
```
*******************************************************************************************************************
## Step 5: Change Java Heap Size

```
vim /opt/jfrog/artifactory/app/bin/artifactory.default --> Just for reference
```
change heap size to 512mb --> if you want play in testing this also just for reference

*******************************************************************************************************************
## Step 6: Install TOMCAT Web Application server
```
mkdir /usr/local/tomcat
sudo wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.71/bin/apache-tomcat-9.0.71.tar.gz
sudo tar xvf apache-tomcat-9.0.71.tar.gz --strip-components=1 -C /usr/local/tomcat
sudo ln -s /usr/local/tomcat/apache-tomcat-9.0.71 /usr/local/tomcat/tomcat
sudo useradd -r tomcat
sudo chown -R tomcat:tomcat /usr/local/tomcat
```

### Configure the tomcat service
```
vim /etc/systemd/system/tomcat.service
```

```
[Unit]
Description=Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment='JAVA_OPTS=-Djava.awt.headless=true'
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINA_BASE=/usr/local/tomcat
Environment=CATALINA_PID=/usr/local/tomcat/temp/tomcat.pid
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
ExecStart=/usr/local/tomcat/bin/catalina.sh start
ExecStop=/usr/local/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
```

save the file and exit.Now run the below commands
--------------------------------------------------------
```
systemctl daemon-reload
systemctl enable tomcat
systemctl start tomcat
systemctl status tomcat
```
--------------------------------------------------------
```
systemctl stop firewalld;systemctl disable firewalld
```
--------------------------------------------------------------------
```
vim /usr/local/tomcat/conf/tomcat-users.xml
```

```
<role rolename="tomcat"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="s3cret" roles="tomcat,manager-gui,manager-script"/>
```


save the file and exit
------------------------------------------------------------
comment the lines in the below file

```
vim /usr/local/tomcat/webapps/manager/META-INF/context.xml
```

```
 <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
	   
```

save the file and exit.Now run the below commands

```
systemctl daemon-reload
systemctl enable tomcat
systemctl start tomcat
systemctl status tomcat
```
-----------------------------------------------------------------------------------------------------
now access the tomcat admin url

- access tomcat console 

http://<linux machine ip address>:8080/
	
- click manager App icon

![image](/images/tomcat-console.png)

- username - tomcat
- password - s3cret


*******************************************************************************************************************
## Step 7: Restart Tomcat and Artifactory Services

```
systemctl restart artifactory;systemctl enable artifactory;systemctl enable tomcat;systemctl restart tomcat
```



http://<linux machine ip address>:8081/artifactory/

*******************************************************************************************************************

## Step 8: Create new pipeline and enable manual deployment to deploy application in tomcat web application server
## Step 9: Create new pipeline and enable auto deployment to deploy application in tomcat web application server
