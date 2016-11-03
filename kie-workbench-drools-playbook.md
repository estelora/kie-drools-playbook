#Linux Install and Configure KIE Workbench and KIE Server

This playbook enables users, developers, and systems people to get started on KIE-workbench and Drools for Centos 6.6.

* Platform: Centos 6 and Wildfly

> Prerequisites: Assumes you are using Linux  (Centos 6.6)
###Grab your software - (Versions are important)! 
####Download and Configure Java JDK 7

* For Java without a yum package, you need a guide: http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/
* After the guide, configure the environmental variables for all users:
 
```
$ sudo sh -c "echo export JAVA_HOME=/opt/jdk1.7.0_79 >> /etc/environment

$ sudo sh -c "echo export JRE_HOME=/opt/jdk1.7.0_79/jre >> /etc/environment

$ sudo sh -c "echo export PATH=$PATH:/opt/jdk1.7.0_79/bin:/opt/jdk1.7.0_79jre/bin >> /etc/environment
```
#### Download the Wildfly Server
```
$ cd /tmp

$ wget http://download.jboss.org/wildfly/8.2.1.Final/wildfly-8.2.1.Final.tar.gz
```  
#### Download KIE Drools Workbench
```
$ cd /tmp

$ wget http://download.jboss.org/drools/release/6.4.0.Final/kie-drools-wb-
   distribution-wars-6.4.0.Final-wildfly8.war
   
```

#### Download KIE Server
```
$ cd /tmp

$ sudo yum install unzip

$ wget http://download.jboss.org/drools/release/6.4.0.Final/kie-server-
  distribution-6.4.0.Final.zip

$ unzip http://download.jboss.org/drools/release/6.4.0.Final/kie-server-
  distribution-6.4.0.Final.zip
```  

 ### Set up Wildfly 8.2.1
 
* Extract wildfly to and move to the `/opt` directory
```
$ tar -xvf wildfly-8.2.1.Final.tar.gz
$ mv wildfly-8.2.1.Final /opt
```
* Add to following lines to the `/opt/wildfly-8.2.1.Final/bin/standalone.conf`:

```
# KIE Server requires full config
export SERVER_CONFIG=--server-config=standalone-full.xml

# find this URL via response from http://localhost:8080/
kie-server-6.4.0.Final-ee7/services/rest/server/
 
export KIE_SERVER_URL=http://localhost:8080/kie-server-6.4.0.Final-ee7/services/rest/server

export KIE_CONTROLLER_URL=http://localhost:8080/kie-drools-wb-distribution-wars-6.4.0.Final-wildfly8/rest/controller

# id must match name of "server template" created in KWB
export KIE_SERVER_ID="-Dorg.kie.server.id=wildfly-kieserver"

# user and pwd must match one of the users created with kie-server role
export KIE_SERVER_USER="-Dorg.kie.server.user=kieserver"
 
export KIE_SERVER_PWD="-Dorg.kie.server.pwd=kieserver"

export KIE_CONTROLLER_USER="-Dorg.kie.server.controller.user=kieserver"

export KIE_CONTROLLER_PWD="-Dorg.kie.server.controller.pwd=kieserver"

export KIE_SERVER="-Dorg.kie.server.location=$KIE_SERVER_URL"

export KIE_CONTROLLER="-Dorg.kie.server.controller=$KIE_CONTROLLER_URL"

export DISABLE_BRM="-Dorg.drools.server.ext.disabled=false"

export DISABLE_BPM="-Dorg.jbpm.server.ext.disabled=true"

# Set kie to ssh_enabled
export KIE_SSH_ENABLED="-Dorg.uberfire.nio.git.ssh.enabled=true"

# Set JBOSS_OPTS 
export JBOSS_OPTS="$SERVER_CONFIG $KIE_SERVER_ID $KIE_SERVER_USER 
  $KIE_SERVER_PWD $KIE_CONTROLLER_USER $KIE_CONTROLLER_PWD $KIE_SERVER 
  $KIE_CONTROLLER $DISABLE_BRM $DISABLE_BPM"

```
### Configure KIE-Workbench 6.4.0

* Move kie-workbench to the `/opt/wildfly-8.2.1.Final/standalone/deployments</code> directory`:
```
$ mv kie-drools-wb-distribution-wars-6.4.0.Final-wildfly8.war /opt/wildfly-8.2.1/standalone/deployments
```
## Set up KIE-Server 6.4.0ee7

* Move the right .war file to the `/opt/wildfly-8.2.1/standalone/deployments</code> directory`:
```
$ cd kie-server-distribution-6.4.0.Final
$ mv kie-server-6.2.0.Final-ee7 /opt/wildfly-8.2.1/standalone/deployments</pre>
```
### Create the wildfly user with appropriate permissions</h2>
```
$ sudo useradd wildfly
$ sudo chown -R wildfly:wildfly /opt/wildfly-8.2.1.Final
```

## Add Wildfly users

* In `/opt/wildfly-8.2.1.Final/bin`, run these two commands to create to create the users (the first is a service account for KIE WorkBench and KIE Server to use with each other; the second is for a human user account):
  * Create the service account:
```
$ cd /opt/wildfly-8.2.1.Final/bin
$ sudo ./add-user.sh -m -u kieserver -p kieserver -ro admin,kie-server,rest-all
```
 * Add your own username(s) and password(s) for the human account:
```
$ sudo ./add-user.sh -a -u wildfly -p wildfly -ro admin,kie-server,rest-all/
```
## Try Starting the server, and check the logs

 * You may have to restart a few times if it times out.
``` 
$ cd /opt/wildfly-8.2.1.Final/bin
$./standalone.sh
```
* Open another terminal window to tail the log:
```
$ tail -f /opt/wildfly-8.2.1.Final/standalone/log/server.log
```
* How to check if your deployment(s) failed:
```
$ cat /opt/wildfly-8.2.1.Final/standalone/deployments/*.failed</pre>
```

### Stop the Server
* Do it from the console:
```
$ su wildlfy
$ cd /opt/wildfly-8.2.1.Final/bin/
$ ./jboss-cli.sh
$ connect
$ shutdown
```
* Exit the window and wait for the log message that it shut down.

### Start the Server
```
$ su wildfly
$ cd /opt/wildfly-8.2.1.Final/bin/
$ ./standalone.sh
```
## Check that all is well with curl, and then you can check the browser, too!
```
$ curl -v --user kieserver:kieserver http://localhost:8080/kie-drools-wb-
  distribution-wars-6.4.0.Final-wildfly8/rest/controller/management/servers
```  
This playbook helps you get started while you work to get things production-ready. If you want to see my drools-kie-workbench chef cookbook, you can check it out <a href="https://github.com/estelora/drools-kie-workbench">here</a>. (This open-source cookbook is not yet production-ready, but it's definitely a great reference if you are considering infrastructure automation for this server).