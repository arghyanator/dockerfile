# Centos Docker Image build on local MAC or Windows 10 

Index
=====

<!--ts-->
   * [MAC](#MAC)
   * [Windows 10](#Windows)
<!--te-->

<a name="MAC"></a>

# Docker for MAC

## Centos 7 Latest derived base image with "systemd"

**Create folder for Base image location on Mac**
```
$ mkdir Docker-images
$ cd Docker-images/
$ mkdir -p local/base
$ cd local/base/
```
**Create Dockerfile for base image**
```
$ vi Dockerfile
```
```
FROM centos:latest
ENV container docker
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
```
```
:wq
```

**Build base image**
```
$ docker build --rm -t local/c7-base .
Sending build context to Docker daemon   2.56kB
Step 1/5 : FROM centos:latest
latest: Pulling from library/centos
7dc0dca2b151: Pull complete 
Digest: sha256:b67d21dfe609ddacf404589e04631d90a342921e81c40aeaf3391f6717fa5322
Status: Downloaded newer image for centos:latest
 ---> 49f7960eb7e4
Step 2/5 : ENV container docker
 ---> Running in bed98f0f3420
Removing intermediate container bed98f0f3420
 ---> b58dc6d357c5
Step 3/5 : RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); rm -f /lib/systemd/system/multi-user.target.wants/*;rm -f /etc/systemd/system/*.wants/*;rm -f /lib/systemd/system/local-fs.target.wants/*; rm -f /lib/systemd/system/sockets.target.wants/*udev*; rm -f /lib/systemd/system/sockets.target.wants/*initctl*; rm -f /lib/systemd/system/basic.target.wants/*;rm -f /lib/systemd/system/anaconda.target.wants/*;
 ---> Running in d5d288728ac0
Removing intermediate container d5d288728ac0
 ---> fec627be5084
Step 4/5 : VOLUME [ "/sys/fs/cgroup" ]
 ---> Running in 1a374922281c
Removing intermediate container 1a374922281c
 ---> 12ac3e684188
Step 5/5 : CMD ["/usr/sbin/init"]
 ---> Running in c1f20b50dcb6
Removing intermediate container c1f20b50dcb6
 ---> aeb8b3df602d
Successfully built aeb8b3df602d
Successfully tagged local/c7-base:latest
```

Check Image
```
$ docker image list |egrep "TAG|c7-base"
REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
local/c7-base                                        latest              aeb8b3df602d        4 minutes ago       200MB
```

## Centos 7 customized image from our base image

**Create Custom image folder**
```
$ mkdir ../custom
$ cd ../custom
```
**Copy required JDK, Tomcat and splunk forwarder files to desktop**
```
####Tomcat 7.0.53
$ wget https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.53/bin/apache-tomcat-7.0.53.tar.gz
--2018-07-25 14:28:40--  https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.53/bin/apache-tomcat-7.0.53.tar.gz
Resolving archive.apache.org (archive.apache.org)... 163.172.17.199
Connecting to archive.apache.org (archive.apache.org)|163.172.17.199|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8780629 (8.4M) [application/x-gzip]
Saving to: ‘apache-tomcat-7.0.53.tar.gz’

apache-tomcat-7.0.53.tar.gz                    100%[====================================================================================================>]   8.37M  2.00MB/s    in 5.0s    

2018-07-25 14:28:46 (1.66 MB/s) - ‘apache-tomcat-7.0.53.tar.gz’ saved [8780629/8780629]

####Oracle JDK jdk1.8.0_51
$ wget https://s3.amazonaws.com/packer-ami-build/packages/jdk-8u51-linux-x64.tar.gz
--2018-07-25 14:46:28--  https://s3.amazonaws.com/packer-ami-build/packages/jdk-8u51-linux-x64.tar.gz
Resolving s3.amazonaws.com (s3.amazonaws.com)... 52.216.64.123
Connecting to s3.amazonaws.com (s3.amazonaws.com)|52.216.64.123|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 173301532 (165M) [application/x-gzip]
Saving to: ‘jdk-8u51-linux-x64.tar.gz’

jdk-8u51-linux-x64.tar.gz                      100%[====================================================================================================>] 165.27M  3.40MB/s    in 85s     

2018-07-25 14:47:54 (1.95 MB/s) - ‘jdk-8u51-linux-x64.tar.gz’ saved [173301532/173301532]

####Splunk Forwarder
$ wget https://s3.amazonaws.com/splunkupgrade/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
--2018-07-25 14:51:15--  https://s3.amazonaws.com/splunkupgrade/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
Resolving s3.amazonaws.com (s3.amazonaws.com)... 52.216.131.189
Connecting to s3.amazonaws.com (s3.amazonaws.com)|52.216.131.189|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 237065943 (226M) [audio/x-pn-realaudio-plugin]
Saving to: ‘splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm’

splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm 100%[====================================================================================================>] 226.08M  1016KB/s    in 2m 25s  

2018-07-25 14:53:41 (1.56 MB/s) - ‘splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm’ saved [237065943/237065943]

```

**Copy Latest application code WAR file**
```
$ wget https://s3.amazonaws.com/software/your_master_linux_201807091734_6dba63dde8d2815.war
--2018-07-31 11:35:38--  https://s3.amazonaws.com/software/your_master_linux_201807091734_6dba63dde8d2815.war
Resolving s3.amazonaws..com (s3.amazonaws.com)... 10.5.101.40, 10.5.101.38
Connecting to s3.amazonaws.com (s3.amazonaws.com)|10.5.101.40|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 76436545 (73M) [application/octet-stream]
Saving to: ‘your_master_linux_201807091734_6dba63dde8d2815.war’
your_master_linux_201807091734_6dba63dde8d2815.war    100%[=================================================================================================================================>]  72.90M  21.8MB/s    in 3.4s    

2018-07-31 11:35:42 (21.3 MB/s) - ‘your_master_linux_201807091734_6dba63dde8d2815.war’ saved [76436545/76436545]
```

**Create tomcat startup file "tomcat"**
```
vi tomcat
```
```
#!/bin/bash
# description: Tomcat Start Stop Restart
# processname: tomcat
# chkconfig: 234 20 80

ulimit -n 65535
JAVA_HOME=/usr/java/jdk1.8.0_51
PATH=$JAVA_HOME/bin:$PATH
CATALINA_HOME=/usr/local/apache-tomcat-7.0.53
export TOMCAT_OPTS=-DENVIRONMENT=local
export ENVIRONMENT=local

export JAVA_HOME
export PATH

case $1 in
start)
JAVA_OPTS="$JAVA_OPTS -Djava.util.logging.config.file=/usr/local/apache-tomcat-7.0.53/conf/logging.properties -DENVIRONMENT=local -DENVIRONMENT_NAME=local -DREGION_NAME=local -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms1024m -Xmx8g -Xss256K -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:ParallelCMSThreads=2 -XX:+CMSClassUnloadingEnabled -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=80  -Dcom.sun.management.jmxremote.port=8050 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.endorsed.dirs=/usr/local/apache-tomcat-7.0.53/endorsed -classpath /usr/local/apache-tomcat-7.0.53/bin/bootstrap.jar:/usr/local/apache-tomcat-7.0.53/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/apache-tomcat-7.0.53 -Dcatalina.home=/usr/local/apache-tomcat-7.0.53 -Djava.io.tmpdir=/usr/local/apache-tomcat-7.0.53/temp"
export JAVA_OPTS
sh $CATALINA_HOME/bin/startup.sh
;;
stop)
sh $CATALINA_HOME/bin/shutdown.sh
;;

esac
exit 0
```
```
:wq
```

**Create Tomcat "server.xml" file**
```
$ vi server.xml
```
```
<?xml version='1.0' encoding='utf-8'?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at
      http://www.apache.org/licenses/LICENSE-2.0
  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<!-- Note:  A "Server" is not itself a "Container", so you may not
     define subcomponents such as "Valves" at this level.
     Documentation at /docs/config/server.html
 -->
<Server port="8005" shutdown="SHUTDOWN">
  <!-- Security listener. Documentation at /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  -->
  <!--APR library loader. Documentation at /docs/apr.html -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <!--Initialize Jasper prior to webapps are loaded. Documentation at /docs/jasper-howto.html -->
  <Listener className="org.apache.catalina.core.JasperListener" />
  <!-- Prevent memory leaks due to use of particular java/javax APIs-->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <!-- Global JNDI resources
       Documentation at /docs/jndi-resources-howto.html
  -->
  <GlobalNamingResources>
    <!-- Editable user database that can also be used by
         UserDatabaseRealm to authenticate users
    -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <!-- A "Service" is a collection of one or more "Connectors" that share
       a single "Container" Note:  A "Service" is not itself a "Container",
       so you may not define subcomponents such as "Valves" at this level.
       Documentation at /docs/config/service.html
   -->
  <Service name="Catalina">

    <!--The connectors can use a shared executor, you can define one or more named thread pools-->
   
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="2000" minSpareThreads="4"/>
   


    <!-- A "Connector" represents an endpoint by which requests are received
         and responses are returned. Documentation at :
         Java HTTP Connector: /docs/config/http.html (blocking & non-blocking)
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         Define a non-SSL HTTP/1.1 Connector on port 8080
    -->
  <!--
  <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxPostSize="67589953" />
 -->
    <!-- A "Connector" using the shared thread pool-->
    
    <Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="300000" acceptCount="500" acceptorThreadCount="2"
               redirectPort="8443" maxPostSize="67589953" />
   
    <!-- Define a SSL HTTP/1.1 Connector on port 8443
         This connector uses the JSSE configuration, when using APR, the
         connector should be using the OpenSSL style configuration
         described in the APR documentation -->
    <!--
    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />
    -->

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />


    <!-- An Engine represents the entry point (within Catalina) that processes
         every request.  The Engine implementation for Tomcat stand alone
         analyzes the HTTP headers included with the request, and passes them
         on to the appropriate Host (virtual host).
         Documentation at /docs/config/engine.html -->

    <!-- You should set jvmRoute to support load-balancing via AJP ie :
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
    -->
    <Engine name="Catalina" defaultHost="localhost">

      <!--For clustering, please take a look at documentation at:
          /docs/cluster-howto.html  (simple how to)
          /docs/config/cluster.html (reference documentation) -->
      <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
      -->

      <!-- Use the LockOutRealm to prevent attempts to guess user passwords
           via a brute-force attack -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- This Realm uses the UserDatabase configured in the global JNDI
             resources under the key "UserDatabase".  Any edits
             that are performed against this UserDatabase are immediately
             available for use by the Realm.  -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" rotatable="false" />

      </Host>
    </Engine>
  </Service>
</Server>
```
```
:wq
```

**Create Startup script file**
```
vi startup.sh
```
```
#!/bin/bash

# Start the Splunk Forwarder
/opt/splunk/bin/splunk start --accept-license && /opt/splunk/bin/splunk enable boot-start &
## for splunk 7.2.1
## /opt/splunk/bin/splunk start --accept-license --no-prompt --answer-yes && /opt/splunk/bin/splunk enable boot-start &
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start Splunk_forwarder: $status"
  exit $status
fi

# Start the Tomcat process
/usr/bin/sudo -u tomcat /etc/init.d/tomcat start &
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start Tomcat: $status"
  exit $status
fi

# Add Tomcat application output logs as input for splunk
sleep 20
/usr/bin/sed -i '/pass4SymmKey =.*$/a allowRemoteLogin = always' /opt/splunk/etc/system/local/server.conf
echo " 
[license]
active_group = Free" >> /opt/splunk/etc/system/local/server.conf
/etc/init.d/splunk restart
sleep 20
/opt/splunk/bin/splunk add monitor /usr/local/apache-tomcat-7.0.53/logs/catalina.out -auth admin:changeme

# Naive check runs checks once a minute to see if either of the processes exited.
# This illustrates part of the heavy lifting you need to do if you want to run
# more than one service in a container. The container exits with an error
# if it detects that either of the processes has exited.
# Otherwise it loops forever, waking up every 60 seconds

while sleep 60; do
  ps aux |grep splunk |grep -q -v grep
  PROCESS_1_STATUS=$?
  ps aux |grep tomcat |grep -q -v grep
  PROCESS_2_STATUS=$?
  # If the greps above find anything, they exit with 0 status
  # If they are not both 0, then something is wrong
  if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 ]; then
  #if [ $PROCESS_2_STATUS -ne 0 ]; then
     echo "One of the processes has already exited."
     exit 1
  fi
done
```
```
:wq
```

> **NOTE:** If Splunk is not required - use the ```startup.sh_noSplunk``` version of the file copied locally as "startup.sh".

**Create Dockerfile**
```
vi Dockerfile
```
```
FROM local/c7-base
MAINTAINER yourname@yourdomain.com
 
#Helpful utils, 
RUN yum -y install vim
RUN yum -y install nc
RUN yum -y install sudo
RUN yum -y install unzip
RUN yum -y install less
RUN yum -y install net-tools
RUN yum -y install firewalld

######## Splunk Forwarder 7.2.1

## REFERENCE https://github.com/splunk/docker-splunk/blob/master/universalforwarder/Dockerfile
## ADD splunk-7.2.1-be11b2c46e23-linux-2.6-x86_64.rpm /tmp/splunk-7.2.1-be11b2c46e23-linux-2.6-x86_64.rpm
## RUN rpm -ihv /tmp/splunk-7.2.1-be11b2c46e23-linux-2.6-x86_64.rpm

######## Splunk Forwarder 6.6.5

## REFERENCE https://github.com/splunk/docker-splunk/blob/master/universalforwarder/Dockerfile
ADD splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
RUN rpm -ihv /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm

########


RUN echo "OPTIMISTIC_ABOUT_FILE_LOCKING = 1" >>/opt/splunk/etc/splunk-launch.conf
EXPOSE 8000
EXPOSE 8089
WORKDIR /opt/splunk
VOLUME [ "/opt/splunk/etc", "/opt/splunk/var"]
 
######## JDK8
 
#Note that ADD uncompresses this tarball automatically
ADD jdk-8u51-linux-x64.tar.gz /usr/java
WORKDIR /usr/java/jdk1.8.0_51
RUN alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_51/bin/java 1
RUN alternatives --install /usr/bin/jar jar /usr/java/jdk1.8.0_51/bin/jar 1
RUN alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_51/bin/javac 1
RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51" >> /etc/environment
 
######## TOMCAT
 
#Note that ADD uncompresses this tarball automatically
ADD apache-tomcat-7.0.53.tar.gz /usr/local
WORKDIR /usr/local
RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51/" >> /etc/default/tomcat7
RUN groupadd tomcat
RUN useradd -s /bin/bash -g tomcat tomcat
RUN chown -Rf tomcat.tomcat /usr/local/apache-tomcat-7.0.53
COPY server.xml /usr/local/apache-tomcat-7.0.53/conf/server.xml
RUN chown tomcat:tomcat /usr/local/apache-tomcat-7.0.53/conf/server.xml
RUN rm -rf /usr/local/apache-tomcat-7.0.53/webapps/ROOT
COPY your_master_linux_201807091734_6dba63dde8d2815.war /usr/local/apache-tomcat-7.0.53/webapps/ROOT.war
RUN unzip -o /usr/local/apache-tomcat-7.0.53/webapps/ROOT.war -d /usr/local/apache-tomcat-7.0.53/webapps/ROOT
# RUN firewall-offline-cmd --add-forward-port=port=80:proto=tcp:toport=8080
EXPOSE 8080
EXPOSE 80

######## Startup Scripts

# Tomcat
COPY tomcat /etc/init.d/tomcat
RUN chmod +x /etc/init.d/tomcat
# Daemon Startup scripts
COPY startup.sh startup.sh
RUN chmod +x startup.sh
CMD ./startup.sh
```
```
:wq
```

**NOTE:** If Splunk is not required - use the ```Dockerfile_noSplunk``` version of the file copied locally as "Dockerfile" (D is upper case).

**Build custom image**
```
$ docker build --rm -t local/c7-custom .
Sending build context to Docker daemon  495.6MB
Step 1/36 : FROM local/c7-base
 ---> aeb8b3df602d
Step 2/36 : MAINTAINER yourname@yourdomain.com
 ---> Running in cb4cab7ef871
Removing intermediate container cb4cab7ef871
 ---> fb07ed53bb96
Step 3/36 : RUN yum -y install vim
 ---> Running in 7bd0811e693c
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: linux.mirrors.es.net
 * extras: centos.mirror.lstn.net
 * updates: mirror.keystealth.org
Resolving Dependencies
--> Running transaction check
---> Package vim-enhanced.x86_64 2:7.4.160-4.el7 will be installed
--> Processing Dependency: vim-common = 2:7.4.160-4.el7 for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: which for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: libperl.so()(64bit) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: libgpm.so.2()(64bit) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Running transaction check
---> Package gpm-libs.x86_64 0:1.20.7-5.el7 will be installed
---> Package perl.x86_64 4:5.16.3-292.el7 will be installed
--> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl-macros for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-292.el7.x86_64
---> Package perl-libs.x86_64 4:5.16.3-292.el7 will be installed
---> Package vim-common.x86_64 2:7.4.160-4.el7 will be installed
--> Processing Dependency: vim-filesystem for package: 2:vim-common-7.4.160-4.el7.x86_64
---> Package which.x86_64 0:2.20-7.el7 will be installed
--> Running transaction check
---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
--> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
--> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
--> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
--> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
---> Package perl-Socket.x86_64 0:2.010-4.el7 will be installed
---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
---> Package perl-macros.x86_64 4:5.16.3-292.el7 will be installed
---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
---> Package vim-filesystem.x86_64 2:7.4.160-4.el7 will be installed
--> Running transaction check
---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
---> Package perl-Pod-Escapes.noarch 1:1.04-292.el7 will be installed
---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
--> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
--> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
--> Running transaction check
---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
--> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: groff-base for package: perl-Pod-Perldoc-3.20-4.el7.noarch
---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
--> Running transaction check
---> Package groff-base.x86_64 0:1.22.2-8.el7 will be installed
---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                     Arch        Version                Repository
                                                                           Size
================================================================================
Installing:
 vim-enhanced                x86_64      2:7.4.160-4.el7        base      1.0 M
Installing for dependencies:
 gpm-libs                    x86_64      1.20.7-5.el7           base       32 k
 groff-base                  x86_64      1.22.2-8.el7           base      942 k
 perl                        x86_64      4:5.16.3-292.el7       base      8.0 M
 perl-Carp                   noarch      1.26-244.el7           base       19 k
 perl-Encode                 x86_64      2.51-7.el7             base      1.5 M
 perl-Exporter               noarch      5.68-3.el7             base       28 k
 perl-File-Path              noarch      2.09-2.el7             base       26 k
 perl-File-Temp              noarch      0.23.01-3.el7          base       56 k
 perl-Filter                 x86_64      1.49-3.el7             base       76 k
 perl-Getopt-Long            noarch      2.40-3.el7             base       56 k
 perl-HTTP-Tiny              noarch      0.033-3.el7            base       38 k
 perl-PathTools              x86_64      3.40-5.el7             base       82 k
 perl-Pod-Escapes            noarch      1:1.04-292.el7         base       51 k
 perl-Pod-Perldoc            noarch      3.20-4.el7             base       87 k
 perl-Pod-Simple             noarch      1:3.28-4.el7           base      216 k
 perl-Pod-Usage              noarch      1.63-3.el7             base       27 k
 perl-Scalar-List-Utils      x86_64      1.27-248.el7           base       36 k
 perl-Socket                 x86_64      2.010-4.el7            base       49 k
 perl-Storable               x86_64      2.45-3.el7             base       77 k
 perl-Text-ParseWords        noarch      3.29-4.el7             base       14 k
 perl-Time-HiRes             x86_64      4:1.9725-3.el7         base       45 k
 perl-Time-Local             noarch      1.2300-2.el7           base       24 k
 perl-constant               noarch      1.27-2.el7             base       19 k
 perl-libs                   x86_64      4:5.16.3-292.el7       base      688 k
 perl-macros                 x86_64      4:5.16.3-292.el7       base       43 k
 perl-parent                 noarch      1:0.225-244.el7        base       12 k
 perl-podlators              noarch      2.5.1-3.el7            base      112 k
 perl-threads                x86_64      1.87-4.el7             base       49 k
 perl-threads-shared         x86_64      1.43-6.el7             base       39 k
 vim-common                  x86_64      2:7.4.160-4.el7        base      5.9 M
 vim-filesystem              x86_64      2:7.4.160-4.el7        base       10 k
 which                       x86_64      2.20-7.el7             base       41 k

Transaction Summary
================================================================================
Install  1 Package (+32 Dependent packages)

Total download size: 19 M
Installed size: 63 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/gpm-libs-1.20.7-5.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for gpm-libs-1.20.7-5.el7.x86_64.rpm is not installed
--------------------------------------------------------------------------------
Total                                              7.9 MB/s |  19 MB  00:02     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.el7.centos.2.x86_64 (@Updates)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:vim-filesystem-7.4.160-4.el7.x86_64                       1/33 
  Installing : 2:vim-common-7.4.160-4.el7.x86_64                           2/33 
  Installing : gpm-libs-1.20.7-5.el7.x86_64                                3/33 
  Installing : groff-base-1.22.2-8.el7.x86_64                              4/33 
  Installing : 1:perl-parent-0.225-244.el7.noarch                          5/33 
  Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                           6/33 
  Installing : perl-podlators-2.5.1-3.el7.noarch                           7/33 
  Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                          8/33 
  Installing : 1:perl-Pod-Escapes-1.04-292.el7.noarch                      9/33 
  Installing : perl-Text-ParseWords-3.29-4.el7.noarch                     10/33 
  Installing : perl-Encode-2.51-7.el7.x86_64                              11/33 
  Installing : perl-Pod-Usage-1.63-3.el7.noarch                           12/33 
  Installing : 4:perl-macros-5.16.3-292.el7.x86_64                        13/33 
  Installing : 4:perl-libs-5.16.3-292.el7.x86_64                          14/33 
  Installing : perl-Storable-2.45-3.el7.x86_64                            15/33 
  Installing : perl-Exporter-5.68-3.el7.noarch                            16/33 
  Installing : perl-constant-1.27-2.el7.noarch                            17/33 
  Installing : perl-Time-Local-1.2300-2.el7.noarch                        18/33 
  Installing : perl-Socket-2.010-4.el7.x86_64                             19/33 
  Installing : perl-Carp-1.26-244.el7.noarch                              20/33 
  Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      21/33 
  Installing : perl-PathTools-3.40-5.el7.x86_64                           22/33 
  Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 23/33 
  Installing : perl-File-Temp-0.23.01-3.el7.noarch                        24/33 
  Installing : perl-File-Path-2.09-2.el7.noarch                           25/33 
  Installing : perl-threads-shared-1.43-6.el7.x86_64                      26/33 
  Installing : perl-threads-1.87-4.el7.x86_64                             27/33 
  Installing : perl-Filter-1.49-3.el7.x86_64                              28/33 
  Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        29/33 
  Installing : perl-Getopt-Long-2.40-3.el7.noarch                         30/33 
  Installing : 4:perl-5.16.3-292.el7.x86_64                               31/33 
  Installing : which-2.20-7.el7.x86_64                                    32/33 
install-info: No such file or directory for /usr/share/info/which.info.gz
  Installing : 2:vim-enhanced-7.4.160-4.el7.x86_64                        33/33 
  Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/33 
  Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       2/33 
  Verifying  : perl-Storable-2.45-3.el7.x86_64                             3/33 
  Verifying  : perl-Exporter-5.68-3.el7.noarch                             4/33 
  Verifying  : perl-constant-1.27-2.el7.noarch                             5/33 
  Verifying  : perl-PathTools-3.40-5.el7.x86_64                            6/33 
  Verifying  : 4:perl-macros-5.16.3-292.el7.x86_64                         7/33 
  Verifying  : 1:perl-parent-0.225-244.el7.noarch                          8/33 
  Verifying  : 4:perl-5.16.3-292.el7.x86_64                                9/33 
  Verifying  : which-2.20-7.el7.x86_64                                    10/33 
  Verifying  : groff-base-1.22.2-8.el7.x86_64                             11/33 
  Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        12/33 
  Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        13/33 
  Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        14/33 
  Verifying  : gpm-libs-1.20.7-5.el7.x86_64                               15/33 
  Verifying  : 4:perl-libs-5.16.3-292.el7.x86_64                          16/33 
  Verifying  : perl-Socket-2.010-4.el7.x86_64                             17/33 
  Verifying  : perl-Carp-1.26-244.el7.noarch                              18/33 
  Verifying  : 2:vim-enhanced-7.4.160-4.el7.x86_64                        19/33 
  Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      20/33 
  Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 21/33 
  Verifying  : 1:perl-Pod-Escapes-1.04-292.el7.noarch                     22/33 
  Verifying  : 2:vim-filesystem-7.4.160-4.el7.x86_64                      23/33 
  Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           24/33 
  Verifying  : perl-Encode-2.51-7.el7.x86_64                              25/33 
  Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         26/33 
  Verifying  : perl-podlators-2.5.1-3.el7.noarch                          27/33 
  Verifying  : perl-File-Path-2.09-2.el7.noarch                           28/33 
  Verifying  : perl-threads-1.87-4.el7.x86_64                             29/33 
  Verifying  : perl-Filter-1.49-3.el7.x86_64                              30/33 
  Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         31/33 
  Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     32/33 
  Verifying  : 2:vim-common-7.4.160-4.el7.x86_64                          33/33 

Installed:
  vim-enhanced.x86_64 2:7.4.160-4.el7                                           

Dependency Installed:
  gpm-libs.x86_64 0:1.20.7-5.el7                                                
  groff-base.x86_64 0:1.22.2-8.el7                                              
  perl.x86_64 4:5.16.3-292.el7                                                  
  perl-Carp.noarch 0:1.26-244.el7                                               
  perl-Encode.x86_64 0:2.51-7.el7                                               
  perl-Exporter.noarch 0:5.68-3.el7                                             
  perl-File-Path.noarch 0:2.09-2.el7                                            
  perl-File-Temp.noarch 0:0.23.01-3.el7                                         
  perl-Filter.x86_64 0:1.49-3.el7                                               
  perl-Getopt-Long.noarch 0:2.40-3.el7                                          
  perl-HTTP-Tiny.noarch 0:0.033-3.el7                                           
  perl-PathTools.x86_64 0:3.40-5.el7                                            
  perl-Pod-Escapes.noarch 1:1.04-292.el7                                        
  perl-Pod-Perldoc.noarch 0:3.20-4.el7                                          
  perl-Pod-Simple.noarch 1:3.28-4.el7                                           
  perl-Pod-Usage.noarch 0:1.63-3.el7                                            
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7                                  
  perl-Socket.x86_64 0:2.010-4.el7                                              
  perl-Storable.x86_64 0:2.45-3.el7                                             
  perl-Text-ParseWords.noarch 0:3.29-4.el7                                      
  perl-Time-HiRes.x86_64 4:1.9725-3.el7                                         
  perl-Time-Local.noarch 0:1.2300-2.el7                                         
  perl-constant.noarch 0:1.27-2.el7                                             
  perl-libs.x86_64 4:5.16.3-292.el7                                             
  perl-macros.x86_64 4:5.16.3-292.el7                                           
  perl-parent.noarch 1:0.225-244.el7                                            
  perl-podlators.noarch 0:2.5.1-3.el7                                           
  perl-threads.x86_64 0:1.87-4.el7                                              
  perl-threads-shared.x86_64 0:1.43-6.el7                                       
  vim-common.x86_64 2:7.4.160-4.el7                                             
  vim-filesystem.x86_64 2:7.4.160-4.el7                                         
  which.x86_64 0:2.20-7.el7                                                     

Complete!
Removing intermediate container 7bd0811e693c
 ---> 2b76fb13fe6e
Step 4/36 : RUN yum -y install nc
 ---> Running in 78d1d40d6bba
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: linux.mirrors.es.net
 * extras: centos.mirror.lstn.net
 * updates: mirror.keystealth.org
Resolving Dependencies
--> Running transaction check
---> Package nmap-ncat.x86_64 2:6.40-13.el7 will be installed
--> Processing Dependency: libpcap.so.1()(64bit) for package: 2:nmap-ncat-6.40-13.el7.x86_64
--> Running transaction check
---> Package libpcap.x86_64 14:1.5.3-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch           Version                    Repository    Size
================================================================================
Installing:
 nmap-ncat         x86_64         2:6.40-13.el7              base         205 k
Installing for dependencies:
 libpcap           x86_64         14:1.5.3-11.el7            base         138 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 343 k
Installed size: 740 k
Downloading packages:
--------------------------------------------------------------------------------
Total                                              1.8 MB/s | 343 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 14:libpcap-1.5.3-11.el7.x86_64                               1/2 
  Installing : 2:nmap-ncat-6.40-13.el7.x86_64                               2/2 
  Verifying  : 14:libpcap-1.5.3-11.el7.x86_64                               1/2 
  Verifying  : 2:nmap-ncat-6.40-13.el7.x86_64                               2/2 

Installed:
  nmap-ncat.x86_64 2:6.40-13.el7                                                

Dependency Installed:
  libpcap.x86_64 14:1.5.3-11.el7                                                

Complete!
Removing intermediate container 78d1d40d6bba
 ---> b2d7b96f43ca
Step 5/36 : RUN yum -y install sudo
 ---> Running in 384d939b6400
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: linux.mirrors.es.net
 * extras: centos.mirror.lstn.net
 * updates: mirror.keystealth.org
Resolving Dependencies
--> Running transaction check
---> Package sudo.x86_64 0:1.8.19p2-14.el7_5 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package      Arch           Version                      Repository       Size
================================================================================
Installing:
 sudo         x86_64         1.8.19p2-14.el7_5            updates         1.1 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 1.1 M
Installed size: 3.9 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : sudo-1.8.19p2-14.el7_5.x86_64                                1/1 
  Verifying  : sudo-1.8.19p2-14.el7_5.x86_64                                1/1 

Installed:
  sudo.x86_64 0:1.8.19p2-14.el7_5                                               

Complete!
Removing intermediate container 384d939b6400
 ---> c5b4fe5d638c
Step 6/36 : RUN yum -y install unzip
 ---> Running in 2320b4e1ca30
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: linux.mirrors.es.net
 * extras: centos.mirror.lstn.net
 * updates: mirror.keystealth.org
Resolving Dependencies
--> Running transaction check
---> Package unzip.x86_64 0:6.0-19.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch              Version               Repository       Size
================================================================================
Installing:
 unzip            x86_64            6.0-19.el7            base            170 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 170 k
Installed size: 365 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : unzip-6.0-19.el7.x86_64                                      1/1 
  Verifying  : unzip-6.0-19.el7.x86_64                                      1/1 

Installed:
  unzip.x86_64 0:6.0-19.el7                                                     

Complete!
Removing intermediate container 2320b4e1ca30
 ---> 2425571cd66a
Step 7/36 : ADD splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
 ---> c2b65137965d
Step 8/36 : RUN rpm -ihv /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
 ---> Running in e830b28f40d7
warning: /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY
Preparing...                          ########################################
Updating / installing...
splunk-6.6.5-b119a2a8b0ad             ########################################
complete
Removing intermediate container e830b28f40d7
 ---> 9f5d828a853b
Step 9/36 : RUN echo "OPTIMISTIC_ABOUT_FILE_LOCKING = 1" >>/opt/splunk/etc/splunk-launch.conf
 ---> Running in 9e407e85398c
Removing intermediate container 9e407e85398c
 ---> 6558a4bb5c3b
Step 10/36 : EXPOSE 8000
 ---> Running in a106cb27b72e
Removing intermediate container a106cb27b72e
 ---> 30ae4694347e
Step 11/36 : EXPOSE 8089
 ---> Running in 4ed5b1a690d5
Removing intermediate container 4ed5b1a690d5
 ---> e7a569f8806c
Step 12/36 : WORKDIR /opt/splunk
Removing intermediate container b02cc3f8dda3
 ---> ceaa5ba1b477
Step 13/36 : VOLUME [ "/opt/splunk/etc", "/opt/splunk/var"]
 ---> Running in 714d73f1d275
Removing intermediate container 714d73f1d275
 ---> 37231aad7483
Step 14/36 : ADD jdk-8u51-linux-x64.tar.gz /usr/java
 ---> 2bed96dd5d9c
Step 15/36 : WORKDIR /usr/java/jdk1.8.0_51
Removing intermediate container bb5ea55efd70
 ---> de28cd56b00a
Step 16/36 : RUN alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_51/bin/java 1
 ---> Running in bd1059b769c5
Removing intermediate container bd1059b769c5
 ---> c48220d3c611
Step 17/36 : RUN alternatives --install /usr/bin/jar jar /usr/java/jdk1.8.0_51/bin/jar 1
 ---> Running in 5d0ca628eb1c
Removing intermediate container 5d0ca628eb1c
 ---> 7131a1ee7423
Step 18/36 : RUN alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_51/bin/javac 1
 ---> Running in 0ae58b3aad5b
Removing intermediate container 0ae58b3aad5b
 ---> 26fe17c7f720
Step 19/36 : RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51" >> /etc/environment
 ---> Running in 852c8e4803a5
Removing intermediate container 852c8e4803a5
 ---> ebdbaf11c0d4
Step 20/36 : ADD apache-tomcat-7.0.53.tar.gz /usr/local
 ---> 894e25fe7f12
Step 21/36 : WORKDIR /usr/local
Removing intermediate container d67cb2e35fb6
 ---> b6d90084dac0
Step 22/36 : RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51/" >> /etc/default/tomcat7
 ---> Running in 7d72c6c02408
Removing intermediate container 7d72c6c02408
 ---> 874321824295
Step 23/36 : RUN groupadd tomcat
 ---> Running in 2cd470b034e4
Removing intermediate container 2cd470b034e4
 ---> 980b776993b6
Step 24/36 : RUN useradd -s /bin/bash -g tomcat tomcat
 ---> Running in 29e00aafdffc
Removing intermediate container 29e00aafdffc
 ---> f4e027d6bed0
Step 25/36 : RUN chown -Rf tomcat.tomcat /usr/local/apache-tomcat-7.0.53
 ---> Running in dcafbd2f7e7d
Removing intermediate container dcafbd2f7e7d
 ---> 6772bcbf1b43
Step 26/36 : COPY server.xml /usr/local/apache-tomcat-7.0.53/conf/server.xml
 ---> d72429bfd333
Step 27/36 : RUN chown tomcat:tomcat /usr/local/apache-tomcat-7.0.53/conf/server.xml
 ---> Running in 87170b3f0028
Removing intermediate container 87170b3f0028
 ---> 535aaf639fed
Step 28/36 : RUN rm -rf /usr/local/apache-tomcat-7.0.53/webapps/ROOT
 ---> Running in 58857bcdb250
Removing intermediate container 58857bcdb250
 ---> 1132eb1baf21
Step 29/36 : COPY your_master_linux_201807091734_6dba63dde8d2815.war /usr/local/apache-tomcat-7.0.53/webapps/ROOT.war
 ---> 40ad4df10c9e
Step 30/36 : RUN unzip -o /usr/local/apache-tomcat-7.0.53/webapps/ROOT.war -d /usr/local/apache-tomcat-7.0.53/webapps/ROOT
 ---> Running in 9bdf9d9ab1c1
Archive:  /usr/local/apache-tomcat-7.0.53/webapps/ROOT.war
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/META-INF/
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/META-INF/MANIFEST.MF  
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-ew1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/com/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-an1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-ue1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-ew1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/local/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-an1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-ue1/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/cd/
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/labs/
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-route53-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-acm-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-logs-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-jaxrs-json-provider-2.4.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-devicefarm-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-common-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-applicationautoscaling-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-autoscaling-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-ecr-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/javax.json-1.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-marketplacecommerceanalytics-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/httpcore-4.4.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/javax.inject-2.4.0-b12.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/protobuf-java-2.6.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-glacier-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-redshift-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-codedeploy-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudwatch-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/slf4j-log4j12-1.7.5.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-elasticache-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cognitosync-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-config-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-api-gateway-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aopalliance-repackaged-2.4.0-b12.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-datapipeline-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/java-dogstatsd-client-2.0.7.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-directconnect-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/bedrock-magma-kinesis-core-1.3.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-rds-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-ses-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/joda-time-2.8.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-dataformat-cbor-2.6.6.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/httpcore-nio-4.4.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jest-common-2.0.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-elastictranscoder-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-emr-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-databind-2.4.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-ec2-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-client-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jest-2.0.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudformation-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-codecommit-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-events-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-importexport-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-elasticsearch-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-workspaces-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-elasticbeanstalk-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/gson-2.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-discovery-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-elasticloadbalancing-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-gamelift-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-efs-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-media-json-jackson-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-lang-2.6.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/validation-api-1.1.0.Final.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/javassist-3.18.1-GA.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-multipart-1.8.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/hk2-api-2.4.0-b12.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-support-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-opsworks-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/httpclient-4.5.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-iot-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-codec-1.2.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-s3-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-server-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-media-multipart-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-io-2.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/osgi-resource-locator-1.0.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-codepipeline-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-lang3-3.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/hk2-utils-2.4.0-b12.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/bedrock-granite-core-0.9.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/mimepull-1.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-simpledb-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/log4j-1.2.17.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-kinesis-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/slf4j-api-1.7.5.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-ssm-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-jaxrs-base-2.5.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-httpclient-3.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-machinelearning-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/hk2-locator-2.4.0-b12.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-swf-libraries-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cognitoidentity-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-lambda-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-core-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-storagegateway-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/argparse4j-0.4.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jsonp-jaxrs-1.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-dms-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-directory-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/guava-18.0.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudsearch-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-entity-filtering-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/mimepull-1.9.5.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-dynamodb-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-sns-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/amazon-kinesis-client-1.6.5.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-module-jaxb-annotations-2.4.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-container-servlet-core-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/javax.ws.rs-api-2.0.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-iam-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudtrail-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-core-1.8.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jsr311-api-1.1.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/javax.annotation-api-1.2.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-bundle-1.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-ecs-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-sqs-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/json-20151123.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-media-json-processing-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-marketplacemeteringservice-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cognitoidp-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-kms-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-waf-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudhsm-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-simpleworkflow-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/httpasyncclient-4.1.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-media-jaxb-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/commons-logging-1.0.4.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudfront-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-cloudwatchmetrics-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-guava-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-inspector-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jersey-container-servlet-2.18.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-core-2.4.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/aws-java-sdk-sts-1.11.14.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/amazon-kinesis-producer-0.12.9.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/lib/jackson-annotations-2.5.1.jar  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/web.xml  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-ew1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-an1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/log4j.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-ue1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/prod-ew1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/local/verifyConfig.json  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/local/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-an1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/stage-ue1/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/cd/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/labs/config.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/example-verifyConfig.json  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/version.properties  
  inflating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/WEB-INF/classes/es-query-ingest-requestid.json  
   creating: /usr/local/apache-tomcat-7.0.53/webapps/ROOT/META-INF/maven/ 
Removing intermediate container 9bdf9d9ab1c1
 ---> 8c1b00294e4a
Step 31/36 : EXPOSE 8080
 ---> Running in 9c95d6d48a45
Removing intermediate container 9c95d6d48a45
 ---> 10e8e6ced9b5
Step 32/36 : COPY tomcat /etc/init.d/tomcat
 ---> c30c45cc4559
Step 33/36 : RUN chmod +x /etc/init.d/tomcat
 ---> Running in 5af7734b5acc
Removing intermediate container 5af7734b5acc
 ---> d41058df3bc1
Step 34/36 : COPY startup.sh startup.sh
 ---> 8ee1ab4e7ec1
Step 35/36 : RUN chmod +x startup.sh
 ---> Running in 8e37bf71f52c
Removing intermediate container 8e37bf71f52c
 ---> dcc220ca8ef3
Step 36/36 : CMD ./startup.sh
 ---> Running in b7ab3f08defd
Removing intermediate container b7ab3f08defd
 ---> 1df976348b85
Successfully built 1df976348b85
Successfully tagged local/c7-custom:latest
```

**Check Docker image**
```
$ docker image list |egrep "TAG|c7"
REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
local/c7-custom                                      latest              d6db764fe742        7 minutes ago       1.73GB
local/c7-base                                        latest              aeb8b3df602d        26 hours ago        200MB
```

**Test our custom container**
```
$ docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 8080:8080 -p 8000:8000 -p 8089:8089 local/c7-custom

This appears to be your first time running this version of Splunk.
Copying '/opt/splunk/etc/openldap/ldap.conf.default' to '/opt/splunk/etc/openldap/ldap.conf'.
Jul 26, 2018 8:34:40 PM org.apache.catalina.core.AprLifecycleListener init
INFO: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
Generating RSA private key, 2048 bit long modulus
..........................+++
..................+++
e is 65537 (0x10001)
writing RSA key

Jul 26, 2018 8:34:40 PM org.apache.coyote.AbstractProtocol init
INFO: Initializing ProtocolHandler ["http-bio-8080"]
Jul 26, 2018 8:34:40 PM org.apache.coyote.AbstractProtocol init
INFO: Initializing ProtocolHandler ["ajp-bio-8009"]
Jul 26, 2018 8:34:40 PM org.apache.catalina.startup.Catalina load
INFO: Initialization processed in 520 ms
Generating RSA private key, 2048 bit long modulus
...............+++
.............................+++
e is 65537 (0x10001)
writing RSA key

Moving '/opt/splunk/share/splunk/search_mrsparkle/modules.new' to '/opt/splunk/share/splunk/search_mrsparkle/modules'.
Jul 26, 2018 8:34:40 PM org.apache.catalina.core.StandardService startInternal
INFO: Starting service Catalina
Jul 26, 2018 8:34:40 PM org.apache.catalina.core.StandardEngine startInternal
INFO: Starting Servlet Engine: Apache Tomcat/7.0.53
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/docs
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/examples
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/manager
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/ROOT
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/host-manager
Jul 26, 2018 8:34:41 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["http-bio-8080"]
Jul 26, 2018 8:34:41 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["ajp-bio-8009"]
Jul 26, 2018 8:34:41 PM org.apache.catalina.startup.Catalina start
INFO: Server startup in 982 ms

Splunk> Map. Reduce. Recycle.

Checking prerequisites...
  Checking http port [8000]: open
  Checking mgmt port [8089]: open
  Checking appserver port [127.0.0.1:8065]: open
  Checking kvstore port [8191]: open
  Checking configuration...  Done.
    Creating: /opt/splunk/var/lib/splunk
    Creating: /opt/splunk/var/run/splunk
    Creating: /opt/splunk/var/run/splunk/appserver/i18n
    Creating: /opt/splunk/var/run/splunk/appserver/modules/static/css
    Creating: /opt/splunk/var/run/splunk/upload
    Creating: /opt/splunk/var/spool/splunk
    Creating: /opt/splunk/var/spool/dirmoncache
    Creating: /opt/splunk/var/lib/splunk/authDb
    Creating: /opt/splunk/var/lib/splunk/hashDb
New certs have been generated in '/opt/splunk/etc/auth'.
  Checking critical directories...  Done
  Checking indexes...
    Validated: _audit _internal _introspection _telemetry _thefishbucket history main summary
  Done


Your license is expired. Please login as an administrator to update the license.

  Checking filesystem compatibility...  Done
  Checking conf files for problems...
  Done
  Checking default conf files for edits...
  Validating installed files against hashes from '/opt/splunk/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64-manifest'
  All installed files intact.
  Done
All preliminary checks passed.

Starting splunk server daemon (splunkd)...  
Generating a 2048 bit RSA private key
............................................+++
......+++
writing new private key to 'privKeySecure.pem'
-----
Signature ok
subject=/CN=11e74683393e/O=SplunkUser
Getting CA Private Key
writing RSA key
Done


Waiting for web server at http://127.0.0.1:8000 to be available... Done


If you get stuck, we're here to help.  
Look for answers here: http://docs.splunk.com

The Splunk web interface is at http://11e74683393e:8000

Init script installed at /etc/init.d/splunk.
Init script is configured to run at boot.
```
**Check Docker container**
```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                        NAMES
09d1efcf068d        local/c7-custom     "/bin/sh -c ./startu…"   44 seconds ago      Up About a minute   8000/tcp, 8089/tcp, 0.0.0.0:8080->8080/tcp   laughing_yonath
```

**Check tomcat URL from MAC**
```
http://localhost:8080/ingest/version?api_key=CreativeCloudSDK
```


**Check the Splunk Free Web UI from MAC**
```
http://localhost:8000
```


**Connect to bash shell on the docker container**
 > Connecting to the bash shell inside the docker container may help during troubleshooting or testing (but does not help in PROD or other environments).

```
$ docker exec -ti $(docker ps |grep local |awk '{print $1}') /bin/bash

# ps -auwx |egrep "tomcat|splunk"

tomcat      23  7.9 35.8 12023320 733244 pts/0 Sl+  17:35   0:25 /usr/java/jdk1.8.0_51/bin/java -Djava.util.logging.config.file=/usr/local/apache-tomcat-7.0.53/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file=/usr/local/apache-tomcat-7.0.53/conf/logging.properties -DENVIRONMENT=local -DENVIRONMENT_NAME=local -DREGION_NAME=local -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms1024m -Xmx8g -Xss256K -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:ParallelCMSThreads=2 -XX:+CMSClassUnloadingEnabled -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=80 -Dcom.sun.management.jmxremote.port=8050 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.endorsed.dirs=/usr/local/apache-tomcat-7.0.53/endorsed -classpath /usr/local/apache-tomcat-7.0.53/bin/bootstrap.jar:/usr/local/apache-tomcat-7.0.53/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/apache-tomcat-7.0.53 -Dcatalina.home=/usr/local/apache-tomcat-7.0.53 -Djava.io.tmpdir=/usr/local/apache-tomcat-7.0.53/temp -Djava.endorsed.dirs=/usr/local/apache-tomcat-7.0.53/endorsed -classpath /usr/local/apache-tomcat-7.0.53/bin/bootstrap.jar:/usr/local/apache-tomcat-7.0.53/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/apache-tomcat-7.0.53 -Dcatalina.home=/usr/local/apache-tomcat-7.0.53 -Djava.io.tmpdir=/usr/local/apache-tomcat-7.0.53/temp org.apache.catalina.startup.Bootstrap start
root       461  1.6  4.5 360768 92144 ?        Sl   17:36   0:04 splunkd -p 8089 restart
root       462  0.0  0.6  71200 14120 ?        Ss   17:36   0:00 [splunkd pid=461] splunkd -p 8089 restart [process-runner]
root       471  1.1  6.5 1361248 133104 ?      Ssl  17:36   0:03 mongod --dbpath=/opt/splunk/var/lib/splunk/kvstore/mongo --port=8191 --timeStampFormat=iso8601-utc --smallfiles --oplogSize=200 --keyFile=/opt/splunk/var/lib/splunk/kvstore/mongo/splunk.key --setParameter=enableLocalhostAuthBypass=0 --replSet=0C1A8DA2-1937-4BC6-8BBA-FAF93E991782 --sslMode=requireSSL --sslAllowInvalidHostnames --sslPEMKeyFile=/opt/splunk/etc/auth/server.pem --sslPEMKeyPassword=xxxxxxxx --sslDisabledProtocols=noTLS1_0,noTLS1_1 --sslCipherConfig=ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDH-ECDSA-AES256-GCM-SHA384:ECDH-ECDSA-AES128-GCM-SHA256:ECDH-ECDSA-AES128-SHA256:AES256-GCM-SHA384:AES128-GCM-SHA256:AES128-SHA256 --nounixsocket --noscripting
root       580  0.7  2.4 1907260 49212 ?       Ssl  17:36   0:02 /opt/splunk/bin/python -O /opt/splunk/lib/python2.7/site-packages/splunk/appserver/mrsparkle/root.py --proxied=127.0.0.1,8065,8000
root       639  0.1  2.2 106044 45804 ?        Ssl  17:36   0:00 /opt/splunk/bin/splunkd instrument-resource-usage -p 8089 --with-kvstore
root       894  0.0  0.0   9096   880 pts/1    S+   17:41   0:00 grep -E --color=auto tomcat|splunk
```

**Stop Container and remove**
```
$ docker stop $(docker ps -a -q); docker rm $(docker ps -a -q)
09d1efcf068d
09d1efcf068d
```

**Delete Docker 'custom' image for further customization**
```
$ docker image list |grep c7
local/c7-custom                                      latest              88aac52b2305        13 minutes ago      2.24GB
local/c7-base                                        latest              aeb8b3df602d        2 weeks ago         200MB
```
Delete the ```local/c7-custom``` container ID.

```
$ docker rmi 88aac52b2305
Untagged: local/c7-custom:latest
Deleted: sha256:88aac52b230516d9d68264dc95dbd72cf7e00ee87a633c62cbc7f33bb39fc0d2
Deleted: sha256:f7cd00e962d4029cb2c416d4c5aab733058722b77c0cb1fb02a2baa708595147
Deleted: sha256:f6f8726aca23992884436a7cf8216a5b7e653eda2b05ad515177ace68f689cde
Deleted: sha256:82d31dd0aefb27ee2fe7f22f99ddc6664a15db68a39e0dd64b4b45495fc24a23
Deleted: sha256:a6096a2aa7e1faff4b86d12a7787ec8c9eb85b8475a3b46410ab476d699efa06
Deleted: sha256:55facd22ab3bdda98c8872d3aadb1a6172a234d3870e9fb404690380d2005bde
Deleted: sha256:02a15320586424d080340f0f728bb5f3282256e2bdaf22fdb1755dd0679c0edc
Deleted: sha256:e4fd3f031806dda30bd121f400d4c61dc3d0d7ac9b369f81e59a234d8eb07fa3
Deleted: sha256:194ef0a9a35df58d2c886adfe2be17a14b03742c9dcd6b294a13242544d57ac6
Deleted: sha256:70a2719b53dcc0cfc6df4dc144cfca7bb78d2bb35c16ee27726d36006b64efd1
Deleted: sha256:07eeeb93e4f8379c009183d69295f558845547e399fb0b36114c36dcaa103a7c
Deleted: sha256:0a72d5282ce87c5e600132f0d6a3fc7f8ff0786eb694c2889ae585ecfc7a38fd
Deleted: sha256:8c21c25fb3439ddb0d57a646161209e7c15b0ad7b37b0cc8eeda3ed47fa9a0c8
Deleted: sha256:d901a43e4f119a64ac295df6172c353d67a2a1fa78df1e48594b75770a31fbff
Deleted: sha256:40ea8d1699ad3fe8e8f5261278e71c88aa9660ba6359ff8d779c791b1375ae95
Deleted: sha256:b64cd72b139406cc9e87f4d8008ddad677a7b002c4bf9408c39098f446e8b95a
Deleted: sha256:ba2396c849099e0099c4f5f44c03e20058da6c2a85223c81a701cf79205747cd
Deleted: sha256:0f9a98805a7a1251fbff018fe44b65a74e384c29122a790b53e4e4b669a5d614
Deleted: sha256:8f1d861b15f4c718f7dd55241eca2eb661eb3f4429a3f6e97b5f2895efb79a13
Deleted: sha256:334c716bb83151540cdde0b99f2e9bb9616f721aa2032d2a9f50d2c82575e0e2
Deleted: sha256:fb8221e08c8fdc24b3a541ae3d0f024ee69b21a65fcd93027ee468347534c8d3
Deleted: sha256:b291df9f0f30902ba1b56ecd11bf96b17426b93bd33fe65a0726d713913be9ce
Deleted: sha256:8334b12f9ab3273143f27d8c07cade33344251aaf8595ecf9f6cf965e8832307
Deleted: sha256:3b51265d507d3480b7201216bd94a1e7033d7b5b0f1ba8ce9eb82db30fe8572b
Deleted: sha256:100e2d0ab448e2d4e3f006c83b93ba4160cfd5643be52c724e5657580e0b2fc5
Deleted: sha256:cba8541489215ccd22d5cfb2174ec0c7d0ba714f93fb73b3ac7d533edd163a87
Deleted: sha256:249612ce9c1e192e5f95a9bbcdc49ae9c2bd4c2a8e0e1080b6d33861c3826de4
Deleted: sha256:c4df5a966c01bfdd8f1665047bb90c8f613796b33aee7c4e4ef8e42b7cd82204
Deleted: sha256:808657138102227fe1d267dd8831ab3b3e91751074b32c3ce6231decdacc842f
Deleted: sha256:c27c8d853d5f0551996d1019cdc9f3d4aef5acd861c046a374faa1a70df57f6d
Deleted: sha256:5b3edbb5d6915c92d31139ee3b36cbfca90a4830543c02f77251c93f19447d5b
Deleted: sha256:fa7322d9991ce0fea2d12241e0178c85477531c83c9c8dc9f29e4b23b3abdb03
Deleted: sha256:8cb1df79f06fe3968dfd2c5dba8100281656b076cd05b563038be27475fdb987
Deleted: sha256:a5f01a034bec00c02de711cffb596c796dd7a549f5bdd28ba924087739f8f1c7
Deleted: sha256:5232a43c3f0621f384491acd2bfff4ac9d9c09d3303ce6e771c19e063c63ec2b
Deleted: sha256:28e914314dd0d47ac080177c3fbe4643c35620ab60e6e79b4f70659a19b70a62
Deleted: sha256:df4c9af853af5886e9cf6e332fce8105c45b469e4b648c1f97fda706a79a16f3
Deleted: sha256:f881dabef945ba32de18a4d97301b497e455b31f205e88432311aded67693ca3
Deleted: sha256:afcdc1f8887fe413f0c2e9fb3311ecbf729c00a9015f2e24e203171a2026d32f
Deleted: sha256:7047baa5b045130cae1746b7daba42e42c76d1bb39d3607e1719ea3125840149
Deleted: sha256:0beaa5e6a13524b478802cb2742c2ae27f26a22b19452c837afc593f66b652df
Deleted: sha256:ee700076bbf26e1380745b879fa66c37eeb7624c7b8f00d857539338f1d5512c
Deleted: sha256:b71ed9bc5d99774f36621b2764f21ae7bcf1bfb17f2b27b9e7089dcfef403f9a
Deleted: sha256:94b2838da06c396c0218532efbe9a5a3ea65ba17a04b95faa9f235ede48d4e54
Deleted: sha256:0fa1fd955a3bb4bba258aa56170d62a8d6afe7d986ca6e9306689c9a8356cf0f
Deleted: sha256:86c199916384e1e77c417a41b2fb84c8e7812189fbda97a11d97071446047601
Deleted: sha256:b0d0802d5a61386eb2b0bf76a34aa4a0444cd7d424f1543ec4c552cc686e69fe
Deleted: sha256:23b9054edcde178172e9da5beec80cf6b6ce3b9b4a74552c932db10c14019708
Deleted: sha256:378c419f5302798d946ec4f0a090a24c858941a7a3e3a317fb09b5e3b3748ea0
Deleted: sha256:6887f114be3a9dd223548ec0b892eeeb53c20a351c890d9f6810144888c3190d
Deleted: sha256:aa7d2102fd5ccc8c13ce6cd43d72d4c514fccb0b1e1b4a927b87ad38247d3568
Deleted: sha256:a25cea3badb29a462226943f692976775d01c5a0fdfd9feaf6743a826a52d192
Deleted: sha256:d7d9e9a9fcbb7933e5f810380ca83dbcb567aa85cb8ffe0375f5f150ffe6b036
Deleted: sha256:5476f5a771ce9e63356c7b14b6ec602fc4d5d519621ee1f710914224dd0dcafe
Deleted: sha256:3c8d1e5ef94928ca616cf272019880656d3bf8fa58f59a5a22a1984448cac204
Deleted: sha256:852660721cac56c3207584175bcf696d9c992ec9733183dded3d8b618edb72d4
Deleted: sha256:8697f5caf0b6a62ece9f89e36305afb40448839f99db516c59b545b6d39c8a6e
Deleted: sha256:ba70ca94112a8464bb51c6f6beb5a82470cfaf241b1c56532aebdbac43cfc8ad
Deleted: sha256:b542f03a0e294c24d1bca1b0ed5874f256c81edd27eb84764a97e3ebfd056e6c
Deleted: sha256:270fe948e1afa811e99eeb1dfec7f7279c5b43f067d72f61e6afbe36618b8e40
Deleted: sha256:841ca1ec4c3f3b1dffab11b0823abca6e7225e5b4a0c82f445a0d0912bf87333
Deleted: sha256:711ba18b490f259067a5761aad7fef073d8ddc5876b200e44898867a9332d4e0
Deleted: sha256:7bf54cf4e1bd9ad1afa9d56ef40a51bead59fd3d3a50d2963446b77bbb53ebd0
Deleted: sha256:b249b5299ecfb1b44c1de2b042f29e6b49d606395ace0d0b439e1b04aec3b57b
Deleted: sha256:7512145bb19e1d1caa7c636e0a48550775d90d3b4d122b8979a7c312849f5473
Deleted: sha256:08df69c07140ee773c880d7bc6bcd56b84ee7d921aa5786fa7dd1a9ef363f865
Deleted: sha256:ac004b723e286668931397ad26317aed49e78604a77b4ad962f2244a3fc757f7
Deleted: sha256:b90e2ee4d4acdf0c1f1329d3bcab613475ab734b38aa88aaf625d49b2a80a259
```

## Run Splunk REST commands using curl
```
$ curl -k -u admin:changeme https://localhost:8089/services/saved/searches \
 -d name=MySavedSearch \
 --data-urlencode search="index=_internal"
 <?xml version="1.0" encoding="UTF-8"?>
<!--This is to override browser formatting; see server.conf[httpServer] to disable. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .-->
<?xml-stylesheet type="text/xml" href="/static/atom.xsl"?>
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:s="http://dev.splunk.com/ns/rest" xmlns:opensearch="http://a9.com/-/spec/opensearch/1.1/">
  <title>savedsearch</title>
  <id>https://localhost:8089/services/saved/searches</id>
  <updated>2018-12-07T23:24:53+00:00</updated>
  <generator build="be11b2c46e23" version="7.2.1"/>
  <author>
    <name>Splunk</name>
  </author>
  <link href="/services/saved/searches/_new" rel="create"/>
  <link href="/services/saved/searches/_reload" rel="_reload"/>
  <link href="/services/saved/searches/_acl" rel="_acl"/>
  <opensearch:totalResults>1</opensearch:totalResults>
  <opensearch:itemsPerPage>30</opensearch:itemsPerPage>
  <opensearch:startIndex>0</opensearch:startIndex>
  <s:messages/>
  <entry>
    <title>MySavedSearch</title>
    <id>https://localhost:8089/servicesNS/admin/search/saved/searches/MySavedSearch</id>
    <updated>2018-12-07T23:24:53+00:00</updated>
    <link href="/servicesNS/admin/search/saved/searches/MySavedSearch" rel="alternate"/>
    <author>
      <name>admin</name>
    </author>
.......
.......
.......
      <s:key name="embed.enabled">0</s:key>
        <s:key name="is_scheduled">0</s:key>
        <s:key name="is_visible">1</s:key>
        <s:key name="max_concurrent">1</s:key>
        <s:key name="next_scheduled_time"></s:key>
        <s:key name="qualifiedSearch">search index=_internal</s:key>
        <s:key name="realtime_schedule">1</s:key>
        <s:key name="request.ui_dispatch_app"></s:key>
        <s:key name="request.ui_dispatch_view"></s:key>
        <s:key name="restart_on_searchpeer_add">1</s:key>
        <s:key name="run_n_times">0</s:key>
        <s:key name="run_on_startup">0</s:key>
        <s:key name="schedule_priority">default</s:key>
        <s:key name="schedule_window">0</s:key>
        <s:key name="search">index=_internal</s:key>
        <s:key name="vsid"></s:key>
        <s:key name="workload_pool"></s:key>
      </s:dict>
    </content>
  </entry>
</feed>
```
<a name="Windows"></a>


# Docker for Windows-10 x-64

## Install Docker on Windows 10

Download docker from - https://download.docker.com/win/stable/InstallDocker.msi

Install using the Docker Documentation at - https://docs.docker.com/docker-for-windows/install/

Also, refer to Microsoft Documentation at - https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10


**Docker Installation tips**

Add Administrator user (or windows user ID that you log in as or your LDAP account) to "docker-users" local group in Windows before you can run Docker on windows.



Run the Docker for windows Shortcut from Desktop to start Docker process (it may enable Hyper-v and restart for the first time its executed)

After Restart - Make sure to start docker for windows process again and make sure its running in task bar before proceeding.

## Enable Virtualization in BIOS of your laptop/desktop

Depending on the make/model of your computer options may be different. For Lenovo X1 Carbon model the way to enable virtualization in BIOS is available at https://amiduos.com/support/knowledge-base/article/enabling-virtualization-in-lenovo-systems <br>

## Running docker

Docker can be started by running the command directly from folder location - "C:\Program Files\Docker\Docker\Docker for Windows.exe"

There are bugs in Docker for windows - but logging off and logging back in and trying again to start the application few times does seem to make docker run.

Once docker is running (notification in windows task bar) - you can run docker cli from "Windows Powershell (admin)".


In power shell run docker CLI 
```
docker ps -l
```

**Create Folder locations**
Create folder where we will store base image and then our custom image (just like Linux above)


**Download the Centos base image**

```
PS C:\Users\yourname\docker-images\local\base> docker pull centos
Using default tag: latest
latest: Pulling from library/centos
256b176beaff: Pull complete
Digest: sha256:6f6d986d425aeabdc3a02cb61c02abb2e78e57357e92417d6d58332856024faf
Status: Downloaded newer image for centos:latest
PS C:\Users\yourname\docker-images\local\base> docker image list
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              5182e96772bf        9 days ago          200MB
```

**Create the Docker file for our base image**

```
PS C:\Users\yourname\docker-images\local\base> notepad .\Dockerfile
```


**NOTE:** Make sure the Dockerfile does not have any ".txt" extension (FILE SHOULD HAVE NO EXTENSION)

Contents of Dockerfile

```
FROM centos

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20180804"

CMD ["/bin/bash"]
```

**Build the Base container image**
```
PS C:\Users\yourname\docker-images\local\base> docker build --rm -t local/c7-base .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM centos
 ---> 5182e96772bf
Step 2/3 : LABEL org.label-schema.schema-version="1.0"     org.label-schema.name="CentOS Base Image"     org.label-schema.vendor="CentOS"     org.label-schema.license="GPLv2"     org.label-schema.build-date="20180804"
 ---> Running in b17287ed83e0
Removing intermediate container b17287ed83e0
 ---> 20023589ae2f
Step 3/3 : CMD ["/bin/bash"]
 ---> Running in e64c59209935
Removing intermediate container e64c59209935
 ---> 4714f61e6d2a
Successfully built 4714f61e6d2a
Successfully tagged local/c7-base:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

Check the image
```
PS C:\Users\yourname\docker-images\local\base> docker image list
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
local/c7-base       latest              4714f61e6d2a        15 seconds ago      200MB
centos              latest              5182e96772bf        9 days ago          200MB
```

**Custom Image**

Create folder for custom image:

```
PS C:\Users\yourname\docker-images\local\base> mkdir ..\custom


    Directory: C:\Users\yourname\docker-images\local


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        8/15/2018   4:50 PM                custom


PS C:\Users\yourname\docker-images\local\base> cd ..\custom\
PS C:\Users\yourname\docker-images\local\custom>
```

Create Dockerfile and startup.sh for custom image (Dockerfile and startup.sh are same for Linux and Windows now)
```
PS C:\Users\yourname\docker-images\local\custom> notepad Dockerfile
PS C:\Users\yourname\docker-images\local\custom> mv .\Dockerfile.txt .\Dockerfile
PS C:\Users\yourname\docker-images\local\custom> dir


    Directory: C:\Users\yourname\docker-images\local\custom


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        8/15/2018   4:52 PM           1518 Dockerfile


PS C:\Users\yourname\docker-images\local\custom> notepad startup.sh
PS C:\Users\yourname\docker-images\local\custom> dir


    Directory: C:\Users\xxxx\docker-images\local\custom


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        8/15/2018   4:52 PM           1518 Dockerfile
-a----        8/15/2018   4:53 PM           1262 startup.sh


PS C:\Users\xxxx\docker-images\local\custom>
```


**NOTE:** If Splunk is not required - use the ```Dockerfile_noSplunk``` version of the file copied locally as "Dockerfile" (D is upper case).
**NOTE:** If Splunk is not required - use the ```startup.sh_noSplunk``` version of the file copied locally as "startup.sh".

> Download all the software to custom folder (Powershell wget is very slow...give it time)
```
PS C:\Users\yourname\docker-images\local\custom> wget https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.53/bin/apache-tomcat-7.0.53.tar.gz -OutFile apache-tomcat-7.0.53.tar.gz

PS C:\Users\yourname\docker-images\local\custom> wget https://s3.amazonaws.com/packer-ami-build/packages/jdk-8u51-linux-x64.tar.gz -OutFile jdk-8u51-linux-x64.tar.gz

PS C:\Users\yourname\docker-images\local\custom> wget https://s3.amazonaws.com/splunkupgrade/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm -OutFile splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
```
Download your Software
```
PS C:\Users\yourname\docker-images\local\custom> wget https://s3.amazonaws.com/software/your/your.war -OutFile your_master_linux_201807091734_6dba63dde8d2815.war
```
Create the tomcat startup script and server.xml configuration files
```
PS C:\Users\xxx\docker-images\local\custom> notepad tomcat
PS C:\Users\xxx\docker-images\local\custom> mv .\tomcat.txt .\tomcat

PS C:\Users\xxxx\docker-images\local\custom> notepad server.xml

PS C:\Users\xxx\docker-images\local\custom> dir


    Directory: C:\Users\xxxx\docker-images\local\custom


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        8/15/2018   4:59 PM        8780629 apache-tomcat-7.0.53.tar.gz
-a----        8/15/2018   4:52 PM           1518 Dockerfile
-a----        8/16/2018  10:44 AM       76436545 your_master_linux_201807091734_6dba63dde8d2815.war
-a----        8/15/2018   5:20 PM      173301532 jdk-8u51-linux-x64.tar.gz
-a----        8/16/2018  11:19 AM           6689 server.xml
-a----        8/16/2018  10:32 AM      237065943 splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
-a----        8/15/2018   4:53 PM           1262 startup.sh
-a----        8/16/2018  11:18 AM           1367 tomcat
```
**Convert all Shell scripts created with Notepad to Unix format**
```
PS C:\Users\xxx\docker-images\local\custom> $txt = (Get-Content -Raw .\startup.sh) -replace "`r`n","`n"
PS C:\Users\xxx\docker-images\local\custom> [io.file]::WriteAllText('C:\Users\xxxx\docker-images\local\custom\startup.unix', $txt)

PS C:\Users\xxxx\docker-images\local\custom> rm .\startup.sh
PS C:\Users\xxxx\docker-images\local\custom> mv .\startup.unix .\startup.sh
```
```
PS C:\Users\xxx\docker-images\local\custom> $txt = (Get-Content -Raw .\tomcat) -replace "`r`n","`n"
PS C:\Users\xxx\docker-images\local\custom> [io.file]::WriteAllText('C:\Users\xxxx\docker-images\local\custom\tomcat.unix', $txt)

PS C:\Users\xxx\docker-images\local\custom> rm .\tomcat
PS C:\Users\xxx\docker-images\local\custom> mv .\tomcat.unix .\tomcat
```
```
PS C:\Users\xxx\docker-images\local\custom> $txt = (Get-Content -Raw .\server.xml) -replace "`r`n","`n"
PS C:\Users\xxx\docker-images\local\custom> [io.file]::WriteAllText('C:\Users\xxxx\docker-images\local\custom\server.unix', $txt)

PS C:\Users\xxx\docker-images\local\custom> rm .\server.xml
PS C:\Users\xxx\docker-images\local\custom> mv .\server.unix .\server.xml
```

**NOTE**

Following Files should exist in the "custom" folder before building container
```
PS C:\Users\xxxx\docker-images\local\custom> dir


    Directory: C:\Users\xxxxx\docker-images\local\custom


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        8/15/2018   4:59 PM        8780629 apache-tomcat-7.0.53.tar.gz
-a----        8/15/2018   4:52 PM           1518 Dockerfile
-a----        8/16/2018  10:44 AM       76436545 your_master_linux_201807091734_6dba63dde8d2815.war
-a----        8/15/2018   5:20 PM      173301532 jdk-8u51-linux-x64.tar.gz
-a----        8/16/2018  11:19 AM           6689 server.xml
-a----        8/16/2018  10:32 AM      237065943 splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
-a----        8/15/2018   4:53 PM           1262 startup.sh
-a----        8/16/2018  11:18 AM           1367 tomcat
```

**Build the "custom" container image with your software and dependencies**
```
PS C:\Users\xxx\docker-images\local\custom> docker build --rm -t local/c7-custom .
Sending build context to Docker daemon  495.6MB
Step 1/28 : FROM local/c7-base
 ---> 4714f61e6d2a
Step 2/28 : MAINTAINER yourname@yourdomain.com
 ---> Running in 07de23ba30b5
Removing intermediate container 07de23ba30b5
 ---> 00e1f80d8b47
Step 3/28 : RUN yum -y install vim
 ---> Running in 01ce3312d6d3
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: mirrors.advancedhosters.com
 * extras: mirrors.tripadvisor.com
 * updates: centos.s.uw.edu
Resolving Dependencies
--> Running transaction check
---> Package vim-enhanced.x86_64 2:7.4.160-4.el7 will be installed
--> Processing Dependency: vim-common = 2:7.4.160-4.el7 for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: which for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: libperl.so()(64bit) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Processing Dependency: libgpm.so.2()(64bit) for package: 2:vim-enhanced-7.4.160-4.el7.x86_64
--> Running transaction check
---> Package gpm-libs.x86_64 0:1.20.7-5.el7 will be installed
---> Package perl.x86_64 4:5.16.3-292.el7 will be installed
--> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl-macros for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-292.el7.x86_64
--> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-292.el7.x86_64
---> Package perl-libs.x86_64 4:5.16.3-292.el7 will be installed
---> Package vim-common.x86_64 2:7.4.160-4.el7 will be installed
--> Processing Dependency: vim-filesystem for package: 2:vim-common-7.4.160-4.el7.x86_64
---> Package which.x86_64 0:2.20-7.el7 will be installed
--> Running transaction check
---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
--> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
--> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
--> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
--> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
---> Package perl-Socket.x86_64 0:2.010-4.el7 will be installed
---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
---> Package perl-macros.x86_64 4:5.16.3-292.el7 will be installed
---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
---> Package vim-filesystem.x86_64 2:7.4.160-4.el7 will be installed
--> Running transaction check
---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
---> Package perl-Pod-Escapes.noarch 1:1.04-292.el7 will be installed
---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
--> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
--> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
--> Running transaction check
---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
--> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
--> Processing Dependency: groff-base for package: perl-Pod-Perldoc-3.20-4.el7.noarch
---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
--> Running transaction check
---> Package groff-base.x86_64 0:1.22.2-8.el7 will be installed
---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                     Arch        Version                Repository
                                                                           Size
================================================================================
Installing:
 vim-enhanced                x86_64      2:7.4.160-4.el7        base      1.0 M
Installing for dependencies:
 gpm-libs                    x86_64      1.20.7-5.el7           base       32 k
 groff-base                  x86_64      1.22.2-8.el7           base      942 k
 perl                        x86_64      4:5.16.3-292.el7       base      8.0 M
 perl-Carp                   noarch      1.26-244.el7           base       19 k
 perl-Encode                 x86_64      2.51-7.el7             base      1.5 M
 perl-Exporter               noarch      5.68-3.el7             base       28 k
 perl-File-Path              noarch      2.09-2.el7             base       26 k
 perl-File-Temp              noarch      0.23.01-3.el7          base       56 k
 perl-Filter                 x86_64      1.49-3.el7             base       76 k
 perl-Getopt-Long            noarch      2.40-3.el7             base       56 k
 perl-HTTP-Tiny              noarch      0.033-3.el7            base       38 k
 perl-PathTools              x86_64      3.40-5.el7             base       82 k
 perl-Pod-Escapes            noarch      1:1.04-292.el7         base       51 k
 perl-Pod-Perldoc            noarch      3.20-4.el7             base       87 k
 perl-Pod-Simple             noarch      1:3.28-4.el7           base      216 k
 perl-Pod-Usage              noarch      1.63-3.el7             base       27 k
 perl-Scalar-List-Utils      x86_64      1.27-248.el7           base       36 k
 perl-Socket                 x86_64      2.010-4.el7            base       49 k
 perl-Storable               x86_64      2.45-3.el7             base       77 k
 perl-Text-ParseWords        noarch      3.29-4.el7             base       14 k
 perl-Time-HiRes             x86_64      4:1.9725-3.el7         base       45 k
 perl-Time-Local             noarch      1.2300-2.el7           base       24 k
 perl-constant               noarch      1.27-2.el7             base       19 k
 perl-libs                   x86_64      4:5.16.3-292.el7       base      688 k
 perl-macros                 x86_64      4:5.16.3-292.el7       base       43 k
 perl-parent                 noarch      1:0.225-244.el7        base       12 k
 perl-podlators              noarch      2.5.1-3.el7            base      112 k
 perl-threads                x86_64      1.87-4.el7             base       49 k
 perl-threads-shared         x86_64      1.43-6.el7             base       39 k
 vim-common                  x86_64      2:7.4.160-4.el7        base      5.9 M
 vim-filesystem              x86_64      2:7.4.160-4.el7        base       10 k
 which                       x86_64      2.20-7.el7             base       41 k

Transaction Summary
================================================================================
Install  1 Package (+32 Dependent packages)

Total download size: 19 M
Installed size: 63 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/gpm-libs-1.20.7-5.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for gpm-libs-1.20.7-5.el7.x86_64.rpm is not installed
--------------------------------------------------------------------------------
Total                                              2.7 MB/s |  19 MB  00:07
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.1.el7.centos.x86_64 (@Updates)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:vim-filesystem-7.4.160-4.el7.x86_64                       1/33
  Installing : 2:vim-common-7.4.160-4.el7.x86_64                           2/33
  Installing : gpm-libs-1.20.7-5.el7.x86_64                                3/33
  Installing : groff-base-1.22.2-8.el7.x86_64                              4/33
  Installing : 1:perl-parent-0.225-244.el7.noarch                          5/33
  Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                           6/33
  Installing : perl-podlators-2.5.1-3.el7.noarch                           7/33
  Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                          8/33
  Installing : 1:perl-Pod-Escapes-1.04-292.el7.noarch                      9/33
  Installing : perl-Text-ParseWords-3.29-4.el7.noarch                     10/33
  Installing : perl-Encode-2.51-7.el7.x86_64                              11/33
  Installing : perl-Pod-Usage-1.63-3.el7.noarch                           12/33
  Installing : 4:perl-macros-5.16.3-292.el7.x86_64                        13/33
  Installing : 4:perl-libs-5.16.3-292.el7.x86_64                          14/33
  Installing : perl-Storable-2.45-3.el7.x86_64                            15/33
  Installing : perl-Exporter-5.68-3.el7.noarch                            16/33
  Installing : perl-constant-1.27-2.el7.noarch                            17/33
  Installing : perl-Time-Local-1.2300-2.el7.noarch                        18/33
  Installing : perl-Socket-2.010-4.el7.x86_64                             19/33
  Installing : perl-Carp-1.26-244.el7.noarch                              20/33
  Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      21/33
  Installing : perl-PathTools-3.40-5.el7.x86_64                           22/33
  Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 23/33
  Installing : perl-File-Temp-0.23.01-3.el7.noarch                        24/33
  Installing : perl-File-Path-2.09-2.el7.noarch                           25/33
  Installing : perl-threads-shared-1.43-6.el7.x86_64                      26/33
  Installing : perl-threads-1.87-4.el7.x86_64                             27/33
  Installing : perl-Filter-1.49-3.el7.x86_64                              28/33
  Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        29/33
  Installing : perl-Getopt-Long-2.40-3.el7.noarch                         30/33
  Installing : 4:perl-5.16.3-292.el7.x86_64                               31/33
  Installing : which-2.20-7.el7.x86_64                                    32/33
install-info: No such file or directory for /usr/share/info/which.info.gz
  Installing : 2:vim-enhanced-7.4.160-4.el7.x86_64                        33/33
  Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/33
  Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       2/33
  Verifying  : perl-Storable-2.45-3.el7.x86_64                             3/33
  Verifying  : perl-Exporter-5.68-3.el7.noarch                             4/33
  Verifying  : perl-constant-1.27-2.el7.noarch                             5/33
  Verifying  : perl-PathTools-3.40-5.el7.x86_64                            6/33
  Verifying  : 4:perl-macros-5.16.3-292.el7.x86_64                         7/33
  Verifying  : 1:perl-parent-0.225-244.el7.noarch                          8/33
  Verifying  : 4:perl-5.16.3-292.el7.x86_64                                9/33
  Verifying  : which-2.20-7.el7.x86_64                                    10/33
  Verifying  : groff-base-1.22.2-8.el7.x86_64                             11/33
  Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        12/33
  Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        13/33
  Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        14/33
  Verifying  : gpm-libs-1.20.7-5.el7.x86_64                               15/33
  Verifying  : 4:perl-libs-5.16.3-292.el7.x86_64                          16/33
  Verifying  : perl-Socket-2.010-4.el7.x86_64                             17/33
  Verifying  : perl-Carp-1.26-244.el7.noarch                              18/33
  Verifying  : 2:vim-enhanced-7.4.160-4.el7.x86_64                        19/33
  Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      20/33
  Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 21/33
  Verifying  : 1:perl-Pod-Escapes-1.04-292.el7.noarch                     22/33
  Verifying  : 2:vim-filesystem-7.4.160-4.el7.x86_64                      23/33
  Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           24/33
  Verifying  : perl-Encode-2.51-7.el7.x86_64                              25/33
  Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         26/33
  Verifying  : perl-podlators-2.5.1-3.el7.noarch                          27/33
  Verifying  : perl-File-Path-2.09-2.el7.noarch                           28/33
  Verifying  : perl-threads-1.87-4.el7.x86_64                             29/33
  Verifying  : perl-Filter-1.49-3.el7.x86_64                              30/33
  Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         31/33
  Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     32/33
  Verifying  : 2:vim-common-7.4.160-4.el7.x86_64                          33/33

Installed:
  vim-enhanced.x86_64 2:7.4.160-4.el7

Dependency Installed:
  gpm-libs.x86_64 0:1.20.7-5.el7
  groff-base.x86_64 0:1.22.2-8.el7
  perl.x86_64 4:5.16.3-292.el7
  perl-Carp.noarch 0:1.26-244.el7
  perl-Encode.x86_64 0:2.51-7.el7
  perl-Exporter.noarch 0:5.68-3.el7
  perl-File-Path.noarch 0:2.09-2.el7
  perl-File-Temp.noarch 0:0.23.01-3.el7
  perl-Filter.x86_64 0:1.49-3.el7
  perl-Getopt-Long.noarch 0:2.40-3.el7
  perl-HTTP-Tiny.noarch 0:0.033-3.el7
  perl-PathTools.x86_64 0:3.40-5.el7
  perl-Pod-Escapes.noarch 1:1.04-292.el7
  perl-Pod-Perldoc.noarch 0:3.20-4.el7
  perl-Pod-Simple.noarch 1:3.28-4.el7
  perl-Pod-Usage.noarch 0:1.63-3.el7
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7
  perl-Socket.x86_64 0:2.010-4.el7
  perl-Storable.x86_64 0:2.45-3.el7
  perl-Text-ParseWords.noarch 0:3.29-4.el7
  perl-Time-HiRes.x86_64 4:1.9725-3.el7
  perl-Time-Local.noarch 0:1.2300-2.el7
  perl-constant.noarch 0:1.27-2.el7
  perl-libs.x86_64 4:5.16.3-292.el7
  perl-macros.x86_64 4:5.16.3-292.el7
  perl-parent.noarch 1:0.225-244.el7
  perl-podlators.noarch 0:2.5.1-3.el7
  perl-threads.x86_64 0:1.87-4.el7
  perl-threads-shared.x86_64 0:1.43-6.el7
  vim-common.x86_64 2:7.4.160-4.el7
  vim-filesystem.x86_64 2:7.4.160-4.el7
  which.x86_64 0:2.20-7.el7

Complete!
Removing intermediate container 01ce3312d6d3
 ---> 8c2e55944d73
Step 4/28 : RUN yum -y install nc
 ---> Running in 759ea08ea369
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: mirrors.advancedhosters.com
 * extras: mirrors.tripadvisor.com
 * updates: centos.s.uw.edu
Resolving Dependencies
--> Running transaction check
---> Package nmap-ncat.x86_64 2:6.40-13.el7 will be installed
--> Processing Dependency: libpcap.so.1()(64bit) for package: 2:nmap-ncat-6.40-13.el7.x86_64
--> Running transaction check
---> Package libpcap.x86_64 14:1.5.3-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch           Version                    Repository    Size
================================================================================
Installing:
 nmap-ncat         x86_64         2:6.40-13.el7              base         205 k
Installing for dependencies:
 libpcap           x86_64         14:1.5.3-11.el7            base         138 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 343 k
Installed size: 740 k
Downloading packages:
--------------------------------------------------------------------------------
Total                                              698 kB/s | 343 kB  00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 14:libpcap-1.5.3-11.el7.x86_64                               1/2
  Installing : 2:nmap-ncat-6.40-13.el7.x86_64                               2/2
  Verifying  : 14:libpcap-1.5.3-11.el7.x86_64                               1/2
  Verifying  : 2:nmap-ncat-6.40-13.el7.x86_64                               2/2

Installed:
  nmap-ncat.x86_64 2:6.40-13.el7

Dependency Installed:
  libpcap.x86_64 14:1.5.3-11.el7

Complete!
Removing intermediate container 759ea08ea369
 ---> 5a0f5628fc81
Step 5/28 : RUN yum -y install sudo
 ---> Running in 491295ce6dbb
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: mirrors.advancedhosters.com
 * extras: mirrors.tripadvisor.com
 * updates: centos.s.uw.edu
Resolving Dependencies
--> Running transaction check
---> Package sudo.x86_64 0:1.8.19p2-14.el7_5 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package      Arch           Version                      Repository       Size
================================================================================
Installing:
 sudo         x86_64         1.8.19p2-14.el7_5            updates         1.1 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 1.1 M
Installed size: 3.9 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : sudo-1.8.19p2-14.el7_5.x86_64                                1/1
  Verifying  : sudo-1.8.19p2-14.el7_5.x86_64                                1/1

Installed:
  sudo.x86_64 0:1.8.19p2-14.el7_5

Complete!
Removing intermediate container 491295ce6dbb
 ---> 341bc8031961
Step 6/28 : ADD splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
 ---> 20717bb04d95
Step 7/28 : RUN rpm -ihv /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm
 ---> Running in f5ee7ee0c12d
warning: /tmp/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY
Preparing...                          ########################################
Updating / installing...
splunk-6.6.5-b119a2a8b0ad             ########################################
complete
Removing intermediate container f5ee7ee0c12d
 ---> 0375d5545c27
Step 8/28 : RUN echo "OPTIMISTIC_ABOUT_FILE_LOCKING = 1" >>/opt/splunk/etc/splunk-launch.conf
 ---> Running in b815f5454227
Removing intermediate container b815f5454227
 ---> a9cba6062050
Step 9/28 : EXPOSE 8000
 ---> Running in 55bbab20a70d
Removing intermediate container 55bbab20a70d
 ---> 93ce1496f47d
Step 10/28 : EXPOSE 8089
 ---> Running in be1bd5ecad55
Removing intermediate container be1bd5ecad55
 ---> f9216295cbea
Step 11/28 : WORKDIR /opt/splunk
 ---> Running in eb2a9c81431c
Removing intermediate container eb2a9c81431c
 ---> c07e0fe4c12c
Step 12/28 : VOLUME [ "/opt/splunk/etc", "/opt/splunk/var"]
 ---> Running in e1536c003951
Removing intermediate container e1536c003951
 ---> 57bc0dfcd42a
Step 13/28 : ADD jdk-8u51-linux-x64.tar.gz /usr/java
 ---> 774c8617e1e0
Step 14/28 : WORKDIR /usr/java/jdk1.8.0_51
 ---> Running in a0b5be9f1a02
Removing intermediate container a0b5be9f1a02
 ---> 31091e1a97f3
Step 15/28 : RUN alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_51/bin/java 1
 ---> Running in 5fc47a13f43f
Removing intermediate container 5fc47a13f43f
 ---> babcb6ade58e
Step 16/28 : RUN alternatives --install /usr/bin/jar jar /usr/java/jdk1.8.0_51/bin/jar 1
 ---> Running in e09c50a87fd4
Removing intermediate container e09c50a87fd4
 ---> 294f6e919cb6
Step 17/28 : RUN alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_51/bin/javac 1
 ---> Running in 398714f1f246
Removing intermediate container 398714f1f246
 ---> 766a916de519
Step 18/28 : RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51" >> /etc/environment
 ---> Running in b8f69bea3051
Removing intermediate container b8f69bea3051
 ---> 6500b864abcf
Step 19/28 : ADD apache-tomcat-7.0.53.tar.gz /usr/local
 ---> 2cb50fc1513d
Step 20/28 : WORKDIR /usr/local
 ---> Running in 57a2176fa169
Removing intermediate container 57a2176fa169
 ---> 82da1b1b3253
Step 21/28 : RUN echo "JAVA_HOME=/usr/java/jdk1.8.0_51/" >> /etc/default/tomcat7
 ---> Running in 90f4fe383ff8
Removing intermediate container 90f4fe383ff8
 ---> 084e1f3fc1a9
Step 22/28 : RUN groupadd tomcat
 ---> Running in bbd221ca5455
Removing intermediate container bbd221ca5455
 ---> 7ed2dbf214ab
Step 23/28 : RUN useradd -s /bin/bash -g tomcat tomcat
 ---> Running in ddde06dbe177
Removing intermediate container ddde06dbe177
 ---> ccd35fac61c0
Step 24/28 : RUN chown -Rf tomcat.tomcat /usr/local/apache-tomcat-7.0.53
 ---> Running in a1459e111c9b
Removing intermediate container a1459e111c9b
 ---> 907a4270c710
Step 25/28 : EXPOSE 8080
 ---> Running in 4c1e89a140f7
Removing intermediate container 4c1e89a140f7
 ---> 77e82d31c65d
Step 26/28 : COPY startup.sh startup.sh
 ---> 34e839a12958
Step 27/28 : RUN chmod +x startup.sh
 ---> Running in d60a83818ac1
Removing intermediate container d60a83818ac1
 ---> 5f822f83cc17
Step 28/28 : CMD ./startup.sh
 ---> Running in ff1df3da4c23
Removing intermediate container ff1df3da4c23
 ---> 9e29212361ad
Successfully built 9e29212361ad
Successfully tagged local/c7-custom:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

Check image registered with Docker

```
PS C:\Users\xxxx\docker-images\local\custom> docker image list
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
local/c7-custom     latest              9e29212361ad        About a minute ago   1.74GB
local/c7-base       latest              4714f61e6d2a        19 hours ago         200MB
centos              latest              5182e96772bf        9 days ago           200MB
```

**Run the container and see if your APP works**

```
PS C:\Users\myname\docker-images\local\custom> docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 8080:8080 -p 8000:8000 -p 8089:8089 local/c7-custom

This appears to be your first time running this version of Splunk.
Aug 16, 2018 6:47:27 PM org.apache.catalina.core.AprLifecycleListener init
INFO: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
Aug 16, 2018 6:47:28 PM org.apache.coyote.AbstractProtocol init
INFO: Initializing ProtocolHandler ["http-bio-8080"]
Aug 16, 2018 6:47:28 PM org.apache.coyote.AbstractProtocol init
INFO: Initializing ProtocolHandler ["ajp-bio-8009"]
Aug 16, 2018 6:47:28 PM org.apache.catalina.startup.Catalina load
INFO: Initialization processed in 2733 ms
Aug 16, 2018 6:47:28 PM org.apache.catalina.core.StandardService startInternal
INFO: Starting service Catalina
Aug 16, 2018 6:47:28 PM org.apache.catalina.core.StandardEngine startInternal
INFO: Starting Servlet Engine: Apache Tomcat/7.0.53
Aug 16, 2018 6:47:28 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/examples
Copying '/opt/splunk/etc/openldap/ldap.conf.default' to '/opt/splunk/etc/openldap/ldap.conf'.
Generating RSA private key, 2048 bit long modulus
...........................................................+++
.....................+++
e is 65537 (0x10001)
writing RSA key

Generating RSA private key, 2048 bit long modulus
....................+++
.......................+++
e is 65537 (0x10001)
writing RSA key

Moving '/opt/splunk/share/splunk/search_mrsparkle/modules.new' to '/opt/splunk/share/splunk/search_mrsparkle/modules'.
Aug 16, 2018 6:47:32 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/manager
Aug 16, 2018 6:47:32 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/host-manager
Aug 16, 2018 6:47:33 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/ROOT
Aug 16, 2018 6:47:33 PM org.apache.catalina.startup.HostConfig deployDirectory
INFO: Deploying web application directory /usr/local/apache-tomcat-7.0.53/webapps/docs
Aug 16, 2018 6:47:33 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["http-bio-8080"]
Aug 16, 2018 6:47:33 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["ajp-bio-8009"]
Aug 16, 2018 6:47:33 PM org.apache.catalina.startup.Catalina start
INFO: Server startup in 5027 ms

Splunk> CSI: Logfiles.

Checking prerequisites...
        Checking http port [8000]: open
        Checking mgmt port [8089]: open
        Checking appserver port [127.0.0.1:8065]: open
        Checking kvstore port [8191]: open
        Checking configuration...  Done.
                Creating: /opt/splunk/var/lib/splunk
                Creating: /opt/splunk/var/run/splunk
                Creating: /opt/splunk/var/run/splunk/appserver/i18n
                Creating: /opt/splunk/var/run/splunk/appserver/modules/static/css
                Creating: /opt/splunk/var/run/splunk/upload
                Creating: /opt/splunk/var/spool/splunk
                Creating: /opt/splunk/var/spool/dirmoncache
                Creating: /opt/splunk/var/lib/splunk/authDb
                Creating: /opt/splunk/var/lib/splunk/hashDb
New certs have been generated in '/opt/splunk/etc/auth'.
        Checking critical directories...        Done
        Checking indexes...
                Validated: _audit _internal _introspection _telemetry _thefishbucket history main summary
        Done


Your license is expired. Please login as an administrator to update the license.

        Checking filesystem compatibility...  Done
        Checking conf files for problems...
        Done
        Checking default conf files for edits...
        Validating installed files against hashes from '/opt/splunk/splunk-6.6.5-b119a2a8b0ad-linux-2.6-x86_64-manifest'
        All installed files intact.
        Done
All preliminary checks passed.

Starting splunk server daemon (splunkd)...
Generating a 2048 bit RSA private key
..................+++
.........+++
writing new private key to 'privKeySecure.pem'
-----
Signature ok
subject=/CN=566f201aeb13/O=SplunkUser
Getting CA Private Key
writing RSA key
Done


Waiting for web server at http://127.0.0.1:8000 to be available............. Done


If you get stuck, we're here to help.
Look for answers here: http://docs.splunk.com

The Splunk web interface is at http://566f201aeb13:8000

Init script installed at /etc/init.d/splunk.
Init script is configured to run at boot.
```

While the container is running - it locks the powershell prompt (not a background process)

**Note:** "Allow Access" to any Windows security warning that pops up for the first time docker container is started.

![tomcat URL])<br>



